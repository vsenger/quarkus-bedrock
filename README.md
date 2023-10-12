# quarkus-bedrock

This project is a sample project to access Amazon Bedrock from a Quarkus application deployed on AWS Lambda as a fat jar app.

The main method is inside the LLMService.java:

```java
public class LLMService {
    //...    

    @GET
    @Path("/ask")
    public String askBedrock(@QueryParam("question") String question) {
        final ObjectMapper objectMapper = new ObjectMapper();
        String response = "";
        try (BedrockRuntimeClient bedrockClient = BedrockRuntimeClient.builder()
                .build()) {
            InvokeModelRequest invokeModelRequest = InvokeModelRequest.builder()
                    .body(SdkBytes.fromString("{\"inputText\" : \"" +
                            question + "\"}", Charset.defaultCharset()))
                    .modelId(llm)
                    .build();

            InvokeModelResponse imResponse = bedrockClient.invokeModel(invokeModelRequest);
            BedrockResult bedrockResult = objectMapper.readValue(
                    imResponse.body().asUtf8String(), BedrockResult.class);
            response = bedrockResult.getResults()[0].getOutputText();
        } catch (Exception e) {
            return "Error" + e.getMessage();
        }
        return "Success: " + response;
    }
    //...
}
```

We will also provide some REST interfaces to check the used llm and change:

```java
public class LLMService {
    //...    
    private static String llm = "amazon.titan-tg1-large";
    @GET
    @Path("/llm")
    public void getLlm(@QueryParam("model") String llmname) {
        llm = llmname;
    }
    @GET
    @Path("/llm_in_use")
    public String getLlm() {
        return llm;
    }
    //...
}

```

## Build & Deploy

Build with:

```shell script
quarkus build --no-tests
```

Deploy it:

```shell script
sam deploy
```

You can proceed with default settings for all options, except for GamingAPI may not have authorization defined, Is this okay?, which must be explicitly answered y.

```
Configuring SAM deploy
======================

	Looking for config file [samconfig.toml] :  Not found

	Setting default arguments for 'sam deploy'
	=========================================
	Stack Name [sam-app]: 
	AWS Region [YOUR-REGION]: 
	#Shows you resources changes to be deployed and require a 'Y' to initiate deploy
	Confirm changes before deploy [y/N]: 
	#SAM needs permission to be able to create roles to connect to the resources in your template
	Allow SAM CLI IAM role creation [Y/n]: 
	#Preserves the state of previously provisioned resources when an operation fails
	Disable rollback [y/N]: 
	GamingAPI may not have authorization defined, Is this okay? [y/N]: y
	Save arguments to configuration file [Y/n]: 
	SAM configuration file [samconfig.toml]: 
	SAM configuration environment [default]: 

	Looking for resources needed for deployment:
	Creating the required resources...

```

You will see the generated output like this:

```
---------------------------------------------------------------------------------
Key                 QuarkusBedrock                                                                                                           
Description         URL for application                                                                                                      
Value               https://someurl.execute-api.us-east-1.amazonaws.com/                                                                  
---------------------------------------------------------------------------------

Successfully created/updated stack - qbr in us-east-1
```

Now you can access your REST interface to Amazon Bedrock:

```
https://someurl.execute-api.us-east-1.amazonaws.com/bedrock/ask?question=Your question to model
```

And you can check the default model:

```
https://someurl.execute-api.us-east-1.amazonaws.com/bedrock/llm_in_use
```

You can change the llm:

```
https://someurl.execute-api.us-east-1.amazonaws.com/bedrock/llm?model=amazon.titan-tg1-large
```



