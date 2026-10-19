check snow flake for s3 only files, send like 10 to high radius via ftp and verify with them that it works, if ftp returns a success response we can use that for updating snowflake as "uploaded" for those SO numbers. the files that we send to high-radius are stored in mysql, the s3 path that is. so it's a pull from snowflake the needed id's then hit our mysql table for the s3 path, then send to high-radius, on success then update snowflake a status.


Ask Scott which doc_type's we want to send