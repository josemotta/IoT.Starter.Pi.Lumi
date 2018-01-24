# IoT Starter Raspberry Pi Lumi

#### Home Intelligence with Raspberry Pi

### An infrared embryo for Home Intelligence using Raspberry Pi with Linux.

## Introduction

This series, targeted to Raspberry Pi with Linux,  started with [IoT.Starter.Pi.Core](https://www.codeproject.com/Articles/1220930/IoT-Starter-Raspberry-Pi-Core) using API First Design strategy  to develop an ASP.NET Core Web Server automatically generated by Swagger Hub.

The second part introduced  [IoT.Starter.Pi.Thing](https://www.codeproject.com/Articles/1224347/IoT-Starter-Raspberry-Pi-Thing), an  embryo for Home Intelligence using Raspberry Pi with Linux. Designed as a starter kit for IoT initiatives, it provides a solid and structured platform to speed up product development. The  [starter kit concept](https://www.codeproject.com/Articles/1122233/IoT-Starter-Core-for-Netduino-Plus) encourages the teams to start working immediately on the product development.

The third part is [IoT.Starter.Pi.Lirc](https://www.codeproject.com/Articles/1226559/IoT-Starter-Raspberry-Pi-Lirc), dedicated to IoT projects that require infrared devices. A test environment is created by a `Console` powered by Lirc, the Linux Infrared Remote Control.  It is now time to benefit from what we learned from these tests.

At this complement of the third part, the objective is to extend the `Thing` web service to allow IR remotes and their respective codes to be considered by the API. Powered by Lirc, the `IoT.Starter.Pi.Lumi` is useful on IoT initiatives that require infrared support.

There will be then two `Things`, available as IoT starter kits:

- **[IoT.Starter.Pi.Thing](https://github.com/josemotta/IoT.Starter.Pi.Thing)**:	an embryo for IoT initiatives, to be used on all projects.

- **[IoT.Starter.Pi.Lumi](https://github.com/josemotta/IoT.Starter.Pi.Lumi)**: an embryo for IoT initiatives powered by Lirc, to be used with infrared (IR) projects.

## API first

Following the API first strategy, a new [API version 1.0.2](https://app.swaggerhub.com/apis/motta/home/1.0.2) is created and summarized below, adding a `RemoteApi` controller to the existing specification. The `GET` operations should identify remotes and their respective IR codes. The `POST` operation reflects the intention to fire IR blasts for remotes already installed at RPI host, using Lirc software.     

![](https://i.imgur.com/Mb5TWpO.png)

### Generate RemoteApi

After we changed the swagger file to Version 1.0.2, SwaggerHub generated automatically the `RemoteApi` controller code. Do you remember "Upgrading the API"? It was explained at [IoT.Starter.Pi.Core](https://www.codeproject.com/Articles/1220930/IoT-Starter-Raspberry-Pi-Core), please take a look there if you have any doubts.

Then, the generated code was tweaked to fit the objective, resulting on the following four operations:

    public class RemoteApiController : Controller
    { 
        /// <summary>
        /// 
        /// </summary>
        /// <remarks>returns ir code from remote</remarks>
        /// <param name="remote">Lirc remote</param>
        /// <param name="code">ir code</param>
        /// <response code="200">All the codes</response>
        [HttpGet]
        [Route("/motta/home/1.0.1/remotes/{remote}/{code}")]
        [ValidateModelState]
        [SwaggerOperation("GetRemoteCode")]
        [SwaggerResponse(200, typeof(List<string>), "All the codes")]
        public virtual IActionResult GetRemoteCode([FromRoute]string remote, [FromRoute]string code)
        {
            string example = ("/usr/bin/irsend list " + remote + " " + code).Bash();

            return new ObjectResult(example);
        }

        /// <summary>
        /// 
        /// </summary>
        /// <remarks>returns all ir codes from remote</remarks>
        /// <param name="remote">Lirc remote</param>
        /// <response code="200">All the codes</response>
        [HttpGet]
        [Route("/motta/home/1.0.1/remotes/{remote}")]
        [ValidateModelState]
        [SwaggerOperation("GetRemoteCodes")]
        [SwaggerResponse(200, typeof(List<string>), "All the codes")]
        public virtual IActionResult GetRemoteCodes([FromRoute]string remote)
        {
            string example = (@"/usr/bin/irsend list " + remote + @" """"").Bash();

            return new ObjectResult(example);
        }

        /// <summary>
        /// 
        /// </summary>
        /// <remarks>returns all installed remotes</remarks>
        /// <param name="skip">number of records to skip</param>
        /// <param name="limit">max number of records to return</param>
        /// <response code="200">All the installed remotes</response>
        [HttpGet]
        [Route("/motta/home/1.0.1/remotes")]
        [ValidateModelState]
        [SwaggerOperation("GetRemotes")]
        [SwaggerResponse(200, typeof(List<string>), "All the installed remotes")]
        public virtual IActionResult GetRemotes([FromQuery]int? skip, [FromQuery]int? limit)
        {
            string example = (@"/usr/bin/irsend list """" """"").Bash();

            return new ObjectResult(example);
        }

        /// <summary>
        /// 
        /// </summary>
        /// <remarks>flashes ir code simulating the remote control</remarks>
        /// <param name="remote">Lirc remote</param>
        /// <param name="code">ir code</param>
        /// <response code="200">response</response>
        [HttpPost]
        [Route("/motta/home/1.0.1/remotes/{remote}/{code}")]
        [ValidateModelState]
        [SwaggerOperation("SendRemoteCode")]
        [SwaggerResponse(200, typeof(ApiResponse), "response")]
        public virtual IActionResult SendRemoteCode([FromRoute]string remote, [FromRoute]string code)
        {
            string example = (@"/usr/bin/irsend send_once " + remote + " " + code).Bash();

            return new ObjectResult(example);
        }
    }
 
You can notice that `irsend` command is used at all them! The `HttpGet` operations identify IR remotes and their respective codes. For example, to list all installed remotes, the following command should be issued by Bash at host:

	pi@lumi:~ $ irsend list "" ""
	
The equivalent code is shown below:

        public virtual IActionResult GetRemotes([FromQuery]int? skip, [FromQuery]int? limit)
        {
            string example = (@"/usr/bin/irsend list """" """"").Bash();

            return new ObjectResult(example);
        }

The `HttpPost` operation has `remote` and `code` parameters, in order to properly blast IR codes through the output IR led. In this case, the `irsend` command at RPI host would be:

	pi@lumi:~ $ irsend SEND_ONCE LED_44_KEY CYAN

The `SendRemoteCode` shows the equivalent below: 

        public virtual IActionResult SendRemoteCode([FromRoute]string remote, [FromRoute]string code)
        {
            string example = (@"/usr/bin/irsend send_once " + remote + " " + code).Bash();

            return new ObjectResult(example);
        }

As already shown at `Lirc-Console`, the `ShellHelper` posted by [loune.net](https://loune.net/2017/06/running-shell-bash-commands-in-net-core/) does the dirty work, starting a bash process, capturing the answer, and returning the string result.

    public static class ShellHelper
    {
        public static string Bash(this string cmd)
        {
            var escapedArgs = cmd.Replace("\"", "\\\"");

            var process = new Process()
            {
                StartInfo = new ProcessStartInfo
                {
                    FileName = "/bin/bash",
                    Arguments = $"-c \"{escapedArgs}\"",
                    RedirectStandardOutput = true,
                    UseShellExecute = false,
                    CreateNoWindow = true,
                }
            };
            process.Start();
            string result = process.StandardOutput.ReadToEnd();
            process.WaitForExit();
            return result;
        }
    }

## `home-web-ir` powered by Lirc

In order to add Lirc support to the project, we apply the same strategy used before, installing Lirc before building the web service. The changes done at dockerfile and docker-compose are listed below.

#### lirc-web.dockerfile

The `lirc-web.dockerfile` shown below use the `dotnet:2.0.0-runtime-stretch-arm32v7` image as base. Before installing Lirc, the OS is updated and upgraded. After Lirc package installation, the RPI configuration is copied to container, assuring that remotes installed at RPI host will be seen. The /boot/config.txt and /etc/lirc/lirc_options.conf are  updated and remote config files are moved to /etc/lirc/lircd.conf.d, in order to keep the same setup installed at RPI host.

	FROM microsoft/dotnet:2.0.0-runtime-stretch-arm32v7 AS base
	ENV DOTNET_CLI_TELEMETRY_OPTOUT 1
	ENV ASPNETCORE_URLS "http://*:5010"
	WORKDIR /app
	
	RUN \
	  apt-get update \
	  && apt-get upgrade -y \
	  && apt-get install -y \
	       lirc \
	  --no-install-recommends && \
	  rm -rf /var/lib/apt/lists/*
	
	RUN \
	  mkdir -p /var/run/lirc \
	  && rm -f /etc/lirc/lircd.conf.d/devinput.*
	
	COPY Lirc/setup/config.txt /boot/config.txt
	COPY Lirc/setup/lirc_options.conf /etc/lirc/lirc_options.conf 
	COPY Lirc/setup/ir-remote.conf /etc/modprobe.d/ir-remote.conf
	COPY Lirc/remotes /etc/lirc/lircd.conf.d
	
	FROM microsoft/dotnet:2.0-sdk AS build
	ENV DOTNET_CLI_TELEMETRY_OPTOUT 1
	ENV ASPNETCORE_URLS "http://*:5010"
	WORKDIR /src
	COPY *.sln ./
	COPY *.dcproj ./
	COPY src/IO.Swagger/IO.Swagger.csproj src/IO.Swagger/
	RUN dotnet restore src/IO.Swagger/
	COPY . .
	WORKDIR /src/src/IO.Swagger
	RUN dotnet build -c Release -r linux-arm -o /app
	
	FROM build AS publish
	RUN dotnet publish -c Release -r linux-arm -o /app
	
	FROM base AS final
	WORKDIR /app
	COPY --from=publish /app .
	ENTRYPOINT ["dotnet", "IO.Swagger.dll"]

Then, same build used at IoT.Starter.Pi.Thing completes the `home-web-ir` building.

#### lirc-compose.yml

The io.swagger service was the only change at `lirc-compose.yml`, as shown below. The image for web service powered by Lirc changed to `home-web-ir`, to differentiate from previous `home-web` with no Lirc support.


	services:
	  io.swagger:
	    container_name: home-web-ir
	    image: josemottalopes/home-web-ir
	    build:
	      context: .
	      dockerfile: Lirc/lirc-web.Dockerfile
	    ports:
	    - "5010"
	    network_mode: bridge
	    privileged: true
	    restart: always
	    volumes:
	    - /var/run/lirc:/var/run/lirc
	    environment:
	      - ASPNETCORE_ENVIRONMENT=Release

A docker volume is created at `/var/run/lirc`, insuring proper communication between containers running `irsend` commands and Lirc output socket installed at RPI host. Since the other services from `IoT.Starter.Pi.Thing` are the same, `home-web` to `home-web-ir` is the only change from `IoT.Starter.Pi.Thing` to `IoT.Starter.Pi.Lumi`.
	
## Running at RPI

Before running the tests, please note that Lirc should be properly installed and configured according to [RPI Setup instructions](https://github.com/josemotta/IoT.Starter.Pi.Lirc/blob/master/RPI_Setup.md). Following is the command to download and run home-web-ir, including  docker volume configuration:

	docker run --privileged -p 5010:5010 -d -it --name=home-web-ir -v /var/run/lirc:/var/run/lirc josemottalopes/home-web-ir:latest

The session below shows the live action.

	root@lumi:~# docker run --privileged -p 5010:5010 -d -it --name=home-web-ir -v /var/run/lirc:/var/run/lirc josemottalopes/home-web-ir:latest
	Unable to find image 'josemottalopes/home-web-ir:latest' locally
	latest: Pulling from josemottalopes/home-web-ir
	0d9fbbfaa2cd: Already exists
	b015fdc7d33a: Already exists
	60aaa226f085: Already exists
	01963091a185: Already exists
	a289a8a5c81a: Already exists
	9432c55829c7: Already exists
	87334e3159b5: Already exists
	08a216585c0f: Pull complete
	ac1e370475ca: Pull complete
	ed21dfb3a130: Pull complete
	885b38cb4c35: Pull complete
	f8526fd2a07f: Pull complete
	Digest: sha256:c374826cfd091ed89fb9da3992bad4d06488fb0152a44d5110401f5ff41de793
	Status: Downloaded newer image for josemottalopes/home-web-ir:latest
	3c11eeecbc26b258e3d57191f2d4ec5d96e60c1bd4478e334da6ebc77a9c222c
	root@lumi:~#

Please check further instructions about [Updating RPI with latest images](https://github.com/josemotta/IoT.Starter.Pi.Lumi/blob/master/Thing_Update.md) in order to optimize your RPI memory.

## Checking results

There are a couple ways to check `home-web-ir` with Internet browser. We can use the "Try it out!" button from Swagger IO and see detailed response, like message body, response code and headers, etc. Another simple way is to  address the service directly at the browser, using the `curl` command or the "Request URL" field provided by Swagger IO.

Following are examples taken from a x64 machine with Windows 10, showing the installed remotes at RPI host.

![](https://i.imgur.com/nyIoLDh.png)

We just need to append the `remote` name to previous address to get its respective IR codes, as shown below.

![](https://i.imgur.com/tOPyoD4.png)

Testing the IR output requires the swagger IO interface, to generate the POST command with "Try it out!" button. You can notice that Samsung monitor answered to "KEY_VOLUMEUP" command as expected.

![](https://i.imgur.com/8RpCoXc.png)

I hope you enjoyed this series, I learned at lot for sure!

Have fun with **[IoT.Starter.Pi.Thing](https://github.com/josemotta/IoT.Starter.Pi.Thing)** and **[IoT.Starter.Pi.Lumi](https://github.com/josemotta/IoT.Starter.Pi.Lumi)** to speed up your IoT initiatives. 

## IOT STARTER KITS

| starter kit  | **[IoT.Starter.Pi.Thing](https://github.com/josemotta/IoT.Starter.Pi.Thing)** | **[IoT.Starter.Pi.Lumi](https://github.com/josemotta/IoT.Starter.Pi.Lumi)** |  
| :---         |     :---:      |          :---: |  
| useful for  | all projects |  infrared (IR) projects |  
| description | embryo for IoT | IR embryo for IoT, powered by Lirc | 
| ssl proxy   | [nginx-proxy](https://hub.docker.com/r/josemottalopes/nginx-proxy/)     | [nginx-proxy](https://hub.docker.com/r/josemottalopes/nginx-proxy/)    |  
| user interface     | [home-ui](https://hub.docker.com/r/josemottalopes/home-ui/)       | [home-ui](https://hub.docker.com/r/josemottalopes/home-ui/)      |  
| web service  | [home-web](https://hub.docker.com/r/josemottalopes/home-web/)       | [home-web-ir](https://hub.docker.com/r/josemottalopes/home-web-ir/)      | 





