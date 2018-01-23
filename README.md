# IoT Starter Raspberry Pi Lumi

#### Home Intelligence with Raspberry Pi

### IoT.Starter.Pi.Thing for a Universal Remote Control

## Introduction

This series, targeted to Raspberry Pi with Linux,  started with [IoT.Starter.Pi.Core](https://www.codeproject.com/Articles/1220930/IoT-Starter-Raspberry-Pi-Core) using API First Design strategy  to develop an ASP.NET Core Web Server automatically generated by Swagger Hub.

The second part introduced  [IoT.Starter.Pi.Thing](https://www.codeproject.com/Articles/1224347/IoT-Starter-Raspberry-Pi-Thing), an  embryo for Home Intelligence using Raspberry Pi with Linux. 
Designed as a starter kit for IoT initiatives, it provides a solid and structured platform to speed up product development. The  [starter kit concept](https://www.codeproject.com/Articles/1122233/IoT-Starter-Core-for-Netduino-Plus) encourages the teams to start working immediately on the product development.

The third part [IoT.Starter.Pi.Lirc](https://www.codeproject.com/Articles/1226559/IoT-Starter-Raspberry-Pi-Lirc) is dedicated to IoT projects that require infrared devices. A test environment is created by a `Console` powered by Lirc, the Linux Infrared Remote Control.  The `Thing` is now able to benefit from what we learned from these tests.

This is a kind of second part of third part. Sorry if you get confused, but the objective now is to extend the `Thing` web services to allow IR remotes and their respective codes to be considered by the API. Powered by Lirc, the `IoT.Starter.Pi.Thing` can be useful on IoT initiatives that require infrared support.

Then we will have two `Things` available as starter kit:

- **[IoT.Starter.Pi.Thing](https://github.com/josemotta/IoT.Starter.Pi.Thing)**:	an embryo for IoT.

- **[IoT.Starter.Pi.Lumi](https://github.com/josemotta/IoT.Starter.Pi.Lumi)**: an embryo for IoT powered by Lirc.

## API first

Following the API first strategy, a new [API version 1.0.2](https://app.swaggerhub.com/apis/motta/home/1.0.2) is created and summarized at picture below, adding a `RemoteApi` controller to existing specification. The `GET` operations should identify remotes and their respective IR codes. The `POST` operation reflects the intention to fire IR blasts for remotes already installed at RPI host, using Lirc software.     

![](https://i.imgur.com/Mb5TWpO.png)

### Generate RemoteApi

SwaggerHub generates automatically the `RemoteApi` controller code, according to instructions already explained at [IoT.Starter.Pi.Core](https://www.codeproject.com/Articles/1220930/IoT-Starter-Raspberry-Pi-Core) in the section "Upgrading the API". Please take a look if you have any doubts. Then the generated code was tweaked to fit our objectives, resulting on the four operations listed below:

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
 
You can notice that `irsend` commands are used in all operations! The `HttpGet` operations identify IR remotes and their respective codes. For example, to list all installed remotes, the following command should be issued at a Bash screen at host:

	pi@lumi:~ $ irsend list "" ""
	
The equivalent code is shown below:

        public virtual IActionResult GetRemotes([FromQuery]int? skip, [FromQuery]int? limit)
        {
            string example = (@"/usr/bin/irsend list """" """"").Bash();

            return new ObjectResult(example);
        }

The `HttpPost` operation get `remote` and `code` parameters to blast the command through the IR output. The Bash command at RPI host would be, for example:

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

## `home-web` powered by Lirc

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

#### lirc-compose.yml

The io.swagger service was the only change at the new `lirc-compose.yml`. A docker volume is created at `/var/run/lirc` insuring the proper communication between containers running `irsend` commands and Lirc output socket installed at RPI host.

	services:
	  io.swagger:
	    container_name: home-web
	    image: josemottalopes/home-web
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
	
