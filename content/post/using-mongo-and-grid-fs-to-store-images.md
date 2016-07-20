+++
date = "2012-08-26T00:00:00"
draft = "true"
title = "Using Mongo and GridFS to store images"
+++
GridFS is Mongo specification to store files, claims to be better than file system but to me querying files it's what makes it lots better and it's very straight forward.

I am using a POJO to use within the service that performs the operations

    public class MongoStoredFile {

        private String id;
        private File file;
        private String name;
        private String contentType;
        private byte[] data;
        private InputStream inputStream;
    }

The service receives a MongoStoredFile object and saves it 

    public String save(MongoStoredFile file) throws IOException {
        GridFS gridFS = new GridFS(mongoTemplate.getDb(),"images");
        GridFSInputFile inputFile = gridFS.createFile(file.getData());
        inputFile.setContentType(file.getContentType());
        inputFile.setFilename(file.getName());
        inputFile.save();
        return inputFile.getId().toString();
    }

it's very straight forward

* create a GridFS object, receives two arguments a connection and a collections name, in this case "images"
* create a GridFSInputFile with files binary data
* set the content type
* set a name
* save it

that simple

Now if you want to retrieve the stored object, here the id is being passed as argument, but you can create a query and handle the results.

    public MongoStoredFile getFile(String objectId) throws IOException  {
        GridFS gridFS = new GridFS(mongoTemplate.getDb(),"images");
        GridFSDBFile file = gridFS.find(new ObjectId(objectId));
        
        ...
    }

* you pass a mongo connection and a collection's name
* execute a query with gridFS.find and you obtain a reference to a GridFSDBFile object 

Now you need to extract the values and put them in our POJO MongoStoredFile

    public MongoStoredFile getFile(String objectId) throws IOException  {
        GridFS gridFS = new GridFS(mongoTemplate.getDb(),"images");
        GridFSDBFile file = gridFS.find(new ObjectId(objectId));
        
        ...
        
        MongoStoredFile mongoStoredFile = new MongoStoredFile();
        mongoStoredFile.setId(objectId.toString());
        mongoStoredFile.setName((String) file.get("filename"));
        mongoStoredFile.setContentType((String) file.get("contentType"));
        mongoStoredFile.setInputStream(file.getInputStream());
        return mongoStoredFile;
    }

that's our service

From the client side we only need to call our service, there's some plumbing to it though

JSP looks like:

    <form:form action="${imagesUpload}" enctype="multipart/form-data" id="imageForm" commandName="mongoStoredFile" name="imageForm" method="POST">
        <fieldset>
            <p>
                <label for="subject">File:</label>
                <form:input path="file" type="file" id="file" 
                            placeholder="File:" required="required" />
            </p>
            <p>
                <input type="submit" id="formButton" value="Upload File" >
            </p>
        </fieldset>
    </form:form>

and the server counterpart

    public void upload(@RequestParam("file") MultipartFile multipartFile, final HttpServletResponse response) throws IOException {
        if (multipartFile == null) {
            throw new RuntimeException("file null");
        }
        MongoStoredFile image = new MongoStoredFile();
        image.setName(multipartFile.getOriginalFilename());
        image.setContentType(multipartFile.getContentType());
        image.setData(multipartFile.getBytes());
        fileService.save(image);
    }

The post would not be complete without an example of a file hosted in this casa. I have taken keen interest in this JVM language for the last month. I am seriously contemplating moving my efforts into that direction.

![Scala](/images/download/503a1791c75e2bb846f5d154)*

You can see the rest of the code at: [https://github.com/lnramirez/lnramirez](https://github.com/lnramirez/lnramirez)

*Image taken from: [http://www.scala-lang.org/docu/files/packageobjects/scala-logo.png](http://www.scala-lang.org/docu/files/packageobjects/scala-logo.png)