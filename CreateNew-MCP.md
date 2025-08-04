# Create a minimal MCP server using C# and publish to NuGet

In this quickstart, you create a minimal Model Context Protocol (MCP) server using the [C# SDK for MCP](https://github.com/modelcontextprotocol/csharp-sdk), connect to it using GitHub Copilot, and publish it to NuGet. MCP servers are services that expose capabilities to clients through the Model Context Protocol (MCP).

> **Note**
> 
> The `Microsoft.Extensions.AI.Templates` experience is currently in preview. The template uses the [ModelContextProtocol](https://www.nuget.org/packages/ModelContextProtocol/) library and the [MCP registry server.json schema](https://github.com/modelcontextprotocol/registry/blob/main/docs/server-json/README.md), which are both in preview.

## Prerequisites

- [.NET 10.0 SDK](https://dotnet.microsoft.com/download/dotnet) (preview 6 or higher)
- [Visual Studio Code](https://code.visualstudio.com/)
- [GitHub Copilot extension](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot) for Visual Studio Code
- [NuGet.org account](https://www.nuget.org/users/account/LogOn)

## Create the project

1. In a terminal window, install the MCP Server template (version 9.7.0-preview.2.25356.2 or newer):

```bash
dotnet new install Microsoft.Extensions.AI.Templates
```

2. Create a new MCP server app with the `dotnet new mcpserver` command:

```bash
dotnet new mcpserver -n SampleMcpServer
```

3. Navigate to the `SampleMcpServer` directory:

```bash
cd SampleMcpServer
```

If .NET SDK is already installed, SKIP this step
Install .NET SDK
> winget install Microsoft.DotNet.SDK.8
> dotnet --version
Update the path if required.


4. Build the project:

```bash
dotnet build
```

5. Update the `<PackageId>` in the `SamplMcpServer.csproj` file to be unique on NuGet.org.
For example `<PackageId>NuGet.org username.SampleMcpServer</PackageId>`.

## Configure the MCP server in Visual Studio Code

Configure GitHub Copilot for Visual Studio Code to use your custom MCP server:

1. If you haven't already, open your project folder in Visual Studio Code.
2. Create a `.vscode` folder at the root of your project.
3. Add an `mcp.json` file in the `.vscode` folder with the following content:

```json
{
  "servers": {
    "SampleMcpServer": {
      "type": "stdio",
      "command": "dotnet",
      "args": [
        "run",
        "--project",
        "<RELATIVE PATH TO PROJECT DIRECTORY>"
      ]
    }
  }
}
```

4. Update the <RELATIVE PATH TO PROJECT DIRECTORY> to "." if the .vscode folder is at the root.
5. Save the file.
6. Click on Start under line 2 of the mcp.json file. 

## Test the MCP server

The MCP server template includes a tool called `get_random_number` you can use for testing and as a starting point for development.

1. Open GitHub Copilot Chat in Visual Studio Code and switch to agent mode.
2. Select the Select tools icon to verify your SampleMcpServer is available with the sample tool listed.
3. Ensure it is selected.
4. Enter a prompt to run the get_random_number tool:

```
Give me a random number between 1 and 100.
```

5. GitHub Copilot requests permission to run the get_random_number tool for your prompt. Select Continue or use the arrow to select a more specific behavior:
   - **Current session** always runs the operation in the current GitHub Copilot Agent Mode session.
   - **Current workspace** always runs the command for the current Visual Studio Code workspace.
   - **Always allow** sets the operation to always run for any GitHub Copilot Agent Mode session or any Visual Studio Code workspace.

6. Verify that the server responds with a random number:

```
Your random number is 42.
```

## Add inputs and configuration options

In this example, you enhance the MCP server to use a configuration value set in an environment variable. This could be configuration needed for the functioning of your MCP server, such as an API key, an endpoint to connect to, or a local directory path.

1. Add another tool method after the `GetRandomNumber` method in `Tools/RandomNumberTools.cs`. Update the tool code to use an environment variable.

```csharp
[McpServerTool]
[Description("Describes random weather in the provided city.")]
public string GetCityWeather(
    [Description("Name of the city to return weather for")] string city)
{
    // Read the environment variable during tool execution.
    // Alternatively, this could be read during startup and passed via IOptions dependency injection
    var weather = Environment.GetEnvironmentVariable("WEATHER_CHOICES");
    if (string.IsNullOrWhiteSpace(weather))
    {
        weather = "balmy,rainy,stormy";
    }

    var weatherChoices = weather.Split(",");
    var selectedWeatherIndex =  Random.Shared.Next(0, weatherChoices.Length);

    return $"The weather in {city} is {weatherChoices[selectedWeatherIndex]}.";
}
```

2. Update the `.vscode/mcp.json` to set the `WEATHER_CHOICES` environment variable for testing.

```json
{
   "servers": {
     "SampleMcpServer": {
       "type": "stdio",
       "command": "dotnet",
       "args": [
         "run",
         "--project",
         "<RELATIVE PATH TO PROJECT DIRECTORY>"
       ],
       "env": {
          "WEATHER_CHOICES": "sunny,humid,freezing"
       }
     }
   }
 }
```

3. Save the file.
4. Click on Start under line 2 of the mcp.json file. 
5. Select the Select tools icon to verify your SampleMcpServer is available with the sample tool listed and Ensure it is selected.
6. Try another prompt with Copilot in VS Code, such as:

```
What is the weather in Redmond, Washington?
```

VS Code should return a random weather description.

4. Update the `.mcp/server.json` to declare your environment variable input.

   - Use the `environment_variables` property to declare environment variables used by your app that will be set by the client using the MCP server (for example, VS Code).
   - Use the `package_arguments` property to define CLI arguments that will be passed to your app. For more examples, see the [MCP Registry project](https://github.com/modelcontextprotocol/registry/blob/main/docs/server-json/examples.md).

```json
{
  "$schema": "https://modelcontextprotocol.io/schemas/draft/2025-07-09/server.json",
  "description": "<your description here>",
  "name": "io.github.<your GitHub username here>/<your repo name>",
  "packages": [
    {
      "registry_name": "nuget",
      "name": "<your package ID here>",
      "version": "<your package version here>",
      "package_arguments": [],
      "environment_variables": [
        {
          "name": "WEATHER_CHOICES",
          "value": "{weather_choices}",
          "variables": {
            "weather_choices": {
              "description": "Comma separated list of weather descriptions to randomly select.",
              "is_required": true,
              "is_secret": false
            }
          }
        }
      ]
    }
  ],
  "repository": {
    "url": "https://github.com/<your GitHub username here>/<your repo name>",
    "source": "github"
  },
  "version_detail": {
    "version": "<your package version here>"
  }
}
```

The only information used by NuGet.org in the `server.json` is the first `packages` array item with the `registry_name` value matching `nuget`. The other top-level properties aside from the `packages` property are currently unused and are intended for the upcoming central MCP Registry. You can leave the placeholder values until the MCP Registry is live and ready to accept MCP server entries.

You can test your MCP server again before moving forward.

## Pack and publish to NuGet

1. Pack the project:

```bash
dotnet pack -c Release
```

2. Publish the package to NuGet:

```bash
dotnet nuget push bin/Release/*.nupkg --api-key <your-api-key> --source https://api.nuget.org/v3/index.json
```

If you want to test the publishing flow before publishing to NuGet.org, you can register an account on the NuGet Gallery integration environment: [https://int.nugettest.org](https://int.nugettest.org/). The `push` command would be modified to:

```bash
dotnet nuget push bin/Release/*.nupkg --api-key <your-api-key> --source https://apiint.nugettest.org/v3/index.json
```

