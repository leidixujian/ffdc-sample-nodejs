### Sample project shows how to obtain security grant (in form of token) from auth server.

1. Create an App in FusionFabric.cloud which includes Financial Toolbox APIs
2. Clone the project.
3. Edit the `.env` file, and enter the client ID, and Access Key. 
4. Install the dependencies and start the server.

```sh
$ npm install
$ npm start
```

Once you started the program, use your favorite browser to go to the following URL: 
http://localhost:5000 to retrieve the output of the call to the **Referential Data** API. 

> To learn how to create this sample project from scratch, follow the tutorial from [sample-express.md](sample-express.md).
