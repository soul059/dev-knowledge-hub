# Handling File Uploads

File uploading is a common requirement in web applications. Servlets provide a straightforward way to handle file uploads using the `@MultipartConfig` annotation and the `Part` interface, which were introduced in Servlet 3.0.

## HTML Form for File Upload

To upload a file, you need an HTML form with the `enctype` attribute set to `multipart/form-data`.

```html
<form action="upload" method="post" enctype="multipart/form-data">
    <input type="file" name="file" />
    <input type="submit" value="Upload" />
</form>
```

## Servlet for File Upload

The servlet that handles the file upload must be annotated with `@MultipartConfig`. This annotation indicates that the servlet expects requests of type `multipart/form-data`.

```java
@WebServlet("/upload")
@MultipartConfig
public class FileUploadServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Part filePart = request.getPart("file");
        String fileName = filePart.getSubmittedFileName();
        
        for (Part part : request.getParts()) {
            part.write("C:\\uploads\\" + fileName);
        }
        
        response.getWriter().print("The file has been uploaded successfully.");
    }
}
```

### Key Concepts

-   **`@MultipartConfig`**: This annotation has several attributes that you can use to configure the file upload, such as `fileSizeThreshold`, `maxFileSize`, and `maxRequestSize`.
-   **`request.getPart("file")`**: This method retrieves the `Part` that corresponds to the file input field in the form.
-   **`part.getSubmittedFileName()`**: This method returns the original file name of the uploaded file.
-   **`part.write("path")`**: This method saves the uploaded file to the specified path.

```