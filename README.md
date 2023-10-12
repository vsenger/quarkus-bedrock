# quarkus-bedrock

This project is a sample project to access Amazon Bedrock from a Quarkus application deployed on AWS Lambda as a fat jar app.

The main method is inside the LLMService.java:

```java
    @GET
    @Path("/ask")
    public String askBedrock(@QueryParam("question") String question) {
        final ObjectMapper objectMapper = new ObjectMapper();
        String response="";
        try (BedrockRuntimeClient bedrockClient = BedrockRuntimeClient.builder()
                .build()) {
            InvokeModelRequest invokeModelRequest = InvokeModelRequest.builder()
                    .body(SdkBytes.fromString("{\"inputText\" : \""+
                            question + "\"}", Charset.defaultCharset()))
                    .modelId(llm)
                    .build();

            InvokeModelResponse imResponse = bedrockClient.invokeModel(invokeModelRequest);
            BedrockResult bedrockResult = objectMapper.readValue(
                    imResponse.body().asUtf8String(), BedrockResult.class);
            response = bedrockResult.getResults()[0].getOutputText();
        }
        catch (Exception e) {
            return "Error" + e.getMessage();
        }
        return "Success: " + response;
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



You can run your application in dev mode that enables live coding using:
```shell script
./mvnw compile quarkus:dev
```

> **_NOTE:_**  Quarkus now ships with a Dev UI, which is available in dev mode only at http://localhost:8080/q/dev/.

## Packaging and running the application

The application can be packaged using:
```shell script
./mvnw package
```
It produces the `quarkus-run.jar` file in the `target/quarkus-app/` directory.
Be aware that it’s not an _über-jar_ as the dependencies are copied into the `target/quarkus-app/lib/` directory.

The application is now runnable using `java -jar target/quarkus-app/quarkus-run.jar`.

If you want to build an _über-jar_, execute the following command:
```shell script
./mvnw package -Dquarkus.package.type=uber-jar
```

The application, packaged as an _über-jar_, is now runnable using `java -jar target/*-runner.jar`.

## Creating a native executable

You can create a native executable using: 
```shell script
./mvnw package -Dnative
```

Or, if you don't have GraalVM installed, you can run the native executable build in a container using: 
```shell script
./mvnw package -Dnative -Dquarkus.native.container-build=true
```

You can then execute your native executable with: `./target/quarkus-bedrock-1.0.0-SNAPSHOT-runner`

If you want to learn more about building native executables, please consult https://quarkus.io/guides/maven-tooling.

## Provided Code

### RESTEasy Reactive

Easily start your Reactive RESTful Web Services

[Related guide section...](https://quarkus.io/guides/getting-started-reactive#reactive-jax-rs-resources)
