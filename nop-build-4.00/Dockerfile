FROM netpotential/msbuild:15.6

MAINTAINER dealdiane@netpotential.co.nz

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

ENV DOTNET_SDK_VERSION=2.1.4 \
    DOTNET_SDK_DOWNLOAD_URL=https://dotnetcli.blob.core.windows.net/dotnet/Sdk/$DOTNET_SDK_VERSION/dotnet-sdk-$DOTNET_SDK_VERSION-win-x64.zip \
    NOP_VERSION=4.00

# .NETCore 2.0
# Install .NET Core SDK
RUN Invoke-WebRequest $Env:DOTNET_SDK_DOWNLOAD_URL -OutFile dotnet.zip; \
    Expand-Archive dotnet.zip -DestinationPath $Env:ProgramFiles\dotnet; \
    Remove-Item -Force dotnet.zip \
    SETX /M PATH $($Env:PATH + ';' + $Env:ProgramFiles + '\dotnet') \
    $env:MSBuildSDKsPath = 'C:\Program Files\dotnet\sdk\' + ${env:DOTNET_SDK_VERSION} + '\Sdks'; \
    [Environment]::SetEnvironmentVariable('MSBuildSDKsPath', $env:MSBuildSDKsPath, [EnvironmentVariableTarget]::Machine);

# Trigger the population of the local package cache
ENV NUGET_XMLDOC_MODE skip
RUN New-Item -Type Directory warmup; \
    cd warmup; \
    dotnet new; \
    cd ..; \
    Remove-Item -Force -Recurse warmup

# Download nopCommerce
RUN [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; \
    Invoke-WebRequest -OutFile C:\7zsetup.exe https://www.7-zip.org/a/7z1801-x64.exe; \
    Start-Process C:\7zsetup.exe -ArgumentList '/S /D=c:\7zip\' -Wait; \
    Invoke-WebRequest "https://github.com/nopSolutions/nopCommerce/releases/download/release-${env:NOP_VERSION}/nopCommerce_${env:NOP_VERSION}_Source.rar" -UseBasicParsing -OutFile "C:/nop-src.rar"; \
    Start-Process C:\7zip\7z.exe -ArgumentList 'x C:/nop-src.rar -oC:/nop' -Wait; \
    Remove-Item -Force "C:\7zsetup.exe"; \
    Remove-Item -Force "C:\nop-src.rar"; \
    Remove-Item -Force -Recurse "C:\7zip";

# Build nopCommerce
# Switch to cmd prompt instead of powershell because passing build options with spaces (e.g. Any Cpu) fails in powershell
SHELL ["cmd", "/c"]
RUN msbuild /p:Configuration=Release /p:Platform="Any CPU" /p:Optimize=true /t:Restore,Build "C:/nop/nopCommerce.sln"

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Workaround for broken node.js https://github.com/nodejs/node/issues/8897
# Projects that require node must be ran from D:\\
VOLUME C:/code
RUN Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\DOS Devices' -Name 'D:' -Value '\??\C:\code' -Type String
WORKDIR 'D:\\'

CMD ["cmd"]