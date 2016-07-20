+++
date = "2014-10-26T00:00:00"
draft = "true"
title = "Groovy scripting"
+++
I fix things now and then, more often tweak HTML and make scripts to do things.

*Dennis Ritchie*


There was a time in my life when the only language I knew was java. One thing that always bothered me was that more often than not just writing the simplest program required too much ceremony. Then I decided that I would learn a scripting language. Python, Ruby, Perl  were on my table, yet again the narrowed java developer mind couldn't adapt so I went for Groovy. Was fairly familiar and had access to all java glory. 

I am no groovy expert and I don't intend to do it. It's the second language I put some effort to learn it but I couldn't master it at all. I do some scripting specially when I need to query databases or create files. 

Cassandra is a NoSQL databases that happens to be very difficult to query since has no relationships, you can say that maybe we are doing things wrong, but I think eventually you'd want to do operations over your data. So one quick way to do it is with groovy. 

It's a very simple script, receives a couple of parameters, connects to cassandra and prints some info to the output in a loop. 

    @Grab('org.slf4j:slf4j-api:1.7.2')
    @Grab('commons-codec:commons-codec:1.9')
    @Grab('com.netflix.astyanax:astyanax-core:1.56.37')
    @Grab('com.netflix.astyanax:astyanax-thrift:1.56.37')
    @Grab('com.netflix.astyanax:astyanax-cassandra:1.56.37')

    import com.netflix.astyanax.*
    import com.netflix.astyanax.impl.AstyanaxConfigurationImpl
    import com.netflix.astyanax.connectionpool.impl.*
    import com.netflix.astyanax.connectionpool.OperationResult
    import com.netflix.astyanax.thrift.ThriftFamilyFactory
    import com.netflix.astyanax.model.ColumnFamily
    import com.netflix.astyanax.connectionpool.exceptions.ConnectionException
    import com.netflix.astyanax.serializers.*
    import java.util.regex.Matcher
    import java.util.regex.Pattern

    def cli = new CliBuilder(usage: 'feed.groovy -[cdflms] [date] [prefix]')

    def formatPattern = 'dd/MM/yyyy HH:mm'

    cli.with {
      h longOpt: 'help', 'Show usage information'
      d longOpt: 'database-file', args: 1, argName: 'file', 'database properties file "file" e.g. local.properties'
      u longOpt: 'user', args: 1, argName: 'user', 'user id'
    }

    def options = cli.parse(args)
    if (!options) {
      return
    }

    if (options.h) {
      cli.usage()
      return
    }

    def now = new Date().format(formatPattern)

    def database = options.d?:'local.properties'

    def userId = options.u

    def props = new Properties()
    new File(database).withInputStream { 
      stream -> props.load(stream) 
    }

    def clusterName = props["cassandra.cluster.name"]
    def keyspaceName = props["cassandra.keyspace.name"]
    def seeds = props["cassandra.seeds"]

    AstyanaxContext context = new AstyanaxContext.Builder().forCluster(clusterName).forKeyspace(keyspaceName)
     .withAstyanaxConfiguration(new AstyanaxConfigurationImpl().setCqlVersion("3.0.0").setTargetCassandraVersion("1.2")
    )
     .withConnectionPoolConfiguration(new ConnectionPoolConfigurationImpl(clusterName+"-"+keyspaceName+"_CONN_POOL")
       .setPort(9160).setSeeds(seeds)
     ).withConnectionPoolMonitor(new CountingConnectionPoolMonitor()).buildKeyspace(ThriftFamilyFactory.getInstance())
     
    context.start()
    Keyspace keyspace = context.getClient()

    def CF_TIMELINE = ColumnFamily.newColumnFamily("timeline", 
        StringSerializer.get(),
        StringSerializer.get())

    OperationResult result

    BufferedReader br = new BufferedReader(new InputStreamReader(System.in))
    def str = 'y'
    while (str.equals('y')) {
      
      result = keyspace.prepareQuery(CF_TIMELINE).withCql("SELECT * FROM timeline WHERE user_id = '${userId}'").execute()
      result.getResult().getRows().each { row ->
        def columns = row.getColumns()
        //do something funny here
      }
      
      str = br.readLine()
    }

This is very simple, the way to run it if you have groovy installed is

    $> groovy cassie.groovy -d load-test.properties -u auserid

Now there will be times when you want to share your script with someone else not initiated in groovy (let's call him a muggle) then how can you wrap this in a brand new jvm without groovy?  To package all this in a java project you'd have to use:
1. create a maven project (gradle is less code)
2. create all the directories that come with that, src, main, resource, com.. blah blah blah
3. set up your pom.xml with some plugin that will package it in one file (maven-one-jar) comes to mind. 
4. by now you are already using an IDE and is no longer a script file, is just a gigantic pain. 

well thank to <http://docs.codehaus.org/display/GROOVY/WrappingGroovyScript> post I was able to compile my script into a single jar **but most importantly** it will fetch also Grapes dependencies.

    /*
     * Copyright 2002-2007 the original author or authors.
     *
     * Licensed under the Apache License, Version 2.0 (the "License");
     * you may not use this file except in compliance with the License.
     * You may obtain a copy of the License at
     *
     *      http://www.apache.org/licenses/LICENSE-2.0
     *
     * Unless required by applicable law or agreed to in writing, software
     * distributed under the License is distributed on an "AS IS" BASIS,
     * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     * See the License for the specific language governing permissions and
     * limitations under the License.
     */

    /**
     * Wrap a script and groovy jars to an executable jar
     */
    def cli = new CliBuilder()
    cli.h( longOpt: 'help', required: false, 'show usage information' )
    cli.d( longOpt: 'destfile', argName: 'destfile', required: false, args: 1, 'jar destintation filename, defaults to {mainclass}.jar' )
    cli.m( longOpt: 'mainclass', argName: 'mainclass', required: true, args: 1, 'fully qualified main class, eg. HelloWorld' )
    cli.c( longOpt: 'groovyc', required: false, 'Run groovyc' )

    //--------------------------------------------------------------------------
    def opt = cli.parse(args)
    if (!opt) { return }
    if (opt.h) {
      cli.usage();
      return
    }

    def mainClass = opt.m
    def scriptBase = mainClass.replace( '.', '/' )
    def scriptFile = new File( scriptBase + '.groovy' )
    if (!scriptFile.canRead()) {
       println "Cannot read script file: '${scriptFile}'"
       return
    }
    def destFile = scriptBase + '.jar'
    if (opt.d) {
      destFile = opt.d
    }

    //--------------------------------------------------------------------------
    def ant = new AntBuilder()

    if (opt.c) {
      ant.echo( "Compiling ${scriptFile}" )
      org.codehaus.groovy.tools.FileSystemCompiler.main( [ scriptFile ] as String[] )
    }

    def GROOVY_HOME = new File( System.getenv('GROOVY_HOME') )
    if (!GROOVY_HOME.canRead()) {
      ant.echo( "Missing environment variable GROOVY_HOME: '${GROOVY_HOME}'" )
      return
    }

    def grapes = [] as HashSet
    ant.echo("Resolving grape dependencies")
    def scriptCode = scriptFile.text
    matcher = (scriptCode =~ /(@Grab\('(.*):(.*):(.*)'\))/)
    matcher.each() { dep ->
      ant.echo("found")
     def uri = groovy.grape.Grape.resolve([autoDownload:true], [groupId: dep[2], artifactId: dep[3], version: dep[4]])
     if (uri?.size()>0) {
      uri.each() { f ->
      ant.echo("Found dependency: ${f.toString()}")
       grapes << f
      }
     }
    } 
     
    ant.jar( destfile: destFile, compress: true, index: true ) {
     fileset( dir: '.', includes: scriptBase + '*.class' )
     zipgroupfileset( dir: GROOVY_HOME, includes: 'embeddable/groovy-all-*.jar' )
     zipgroupfileset( dir: GROOVY_HOME, includes: 'lib/commons*.jar' )
     grapes.each() { grape ->
       def grapeFile = new File(grape)
       if (grapeFile.exists()) {
           zipgroupfileset(dir: grapeFile.parent, includes: grapeFile.name)
       }
     }
     manifest {
        attribute( name: 'Main-Class', value: mainClass )
      }
    }

    ant.echo( "Run script using: \'java -jar ${destFile} ...\'" )

then in order to do all the packaging you do

    groovy GroovyWrapper -c -m cassie

this will create a jar *cassie.jar* 

and to run it

    java -Dgroovy.grape.enable=false -jar cassie.jar -d load-test.properties -u aUserId

voila you have a quick way to do scripts that use the good java libraries you love, in a single file and if you want to share it with muggles is very simple too. 