# Performing an API Deploy With No Delay

Here we will show you the steps to perform an API deployment with no delay, using Azure.


## 🛠 Technologies Used

- Visual Studio;

- .NET

- Azure Microsoft Functions

- Azure DevOps



## 🚀 Steps

- First we need to create the Web API, for this we use Visual Studio;
  
- We also need to create a project in Azure DevOps to create our PipeLine;
  
- We need to ajust some file paths on Dokerfile:

```csharp

# This stage is used to build the service project
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["APITempoDIO.csproj", "./"]
RUN dotnet restore "APITempoDIO.csproj"
COPY . .
WORKDIR "/src"
RUN dotnet build "APITempoDIO.csproj" -c $BUILD_CONFIGURATION -o /app/build

# This stage is used to publish the service project to be copied to the final stage
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "APITempoDIO.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

# This stage is used in production or when running from VS in regular mode (Default when not using the debugger)
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "APITempoDIO.dll"]

```


- Do the fisrt commit:

```sh

git add .
git commit -m "First Commit"
git push

```


- Now that we have created the API, we need to create the Container Registry and the Web App via Microsoft Azure App;

- we need to configure the PipeLine on Azure DevOps, we teste this steps using the configs below, but you cam try differents ones;

```yaml

# Starter pipeline
# Start with a minimal pipeline that you can customize to build and deploy your code.
# Add steps that build, run tests, deploy, and more:
# https://aka.ms/yaml

trigger:
- main

pool:
  vmImage: ubuntu-latest

variables:
  - solution: '**/*.sln'
  - buildPlatform: 'Any CPU'
  - buildConfiguration: 'Release'

steps:
- task: UseDotNet@2
  displayName: 'Install .Net SDK'
  inputs:
    packageType: 'sdk'
    version: '8.x'

- script: dotnet restore $(solution)
  displayName: 'Restore Solution'

- script: dotnet build $(solution) --configuration $(buildConfiguration)
  displayName: 'Build Solution'

- script: dotnet test $(solution) --configuration $(buildConfiguration) --no-build --collect:"XPlat Code Coverage"
  displayName: 'Run Tests'

- task: Docker@2
  inputs:
    containerRegistry: 'acrapidemohsouza'
    repository: 'api-dio-test'
    command: 'buildAndPush'
    Dockerfile: './APITempoDIO/APITempoDIO/Dockerfile'

- script: echo Hello, world!
  displayName: 'Run a one-line script'

- script: |
    echo Add other tasks to build, test, and deploy your project.
    echo See https://aka.ms/yaml
  displayName: 'Run a multi-line script'

```


- Now we need to config and creat the WebHook to the API. But first we need to use the update admin comand:

```sh

az acr -n <acrName> --admin-enabled true

```


- Go to Web App on Microsoft Azure, configure the basic credentials and configure the Deployment Center.

- Ajust the trigger option on Azure DevOps, selecting the option "Override the YAML continous integration trigger from here" to automate the process;

- You need to do a ```sh git pull``` on Visual Studio before execute the API.

- Then, do some changes on the API;

- Commit the changes:

```sh
git add .
git commit -m "New Commit"
git push

```

- You can see that PipeLine automatically applies the changes and adds the new API version. If you configure the latest version always set. It will automatically deploy the new version. Otherwise, you need to switch to the new API version manually with Microsoft Azure. But in both cases, the debloy will be done without any API delay. The API remains up all the time.
