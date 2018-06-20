FROM netpotential/msbuild:15.6

MAINTAINER dealdiane@netpotential.co.nz
    
ENV NOP_VERSION 3.80

# Download nopCommerce
RUN [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; \
    Invoke-WebRequest -OutFile C:\7zsetup.exe https://www.7-zip.org/a/7z1801-x64.exe; \
    Start-Process C:\7zsetup.exe -ArgumentList '/S /D=c:\7zip\' -Wait; \
    Invoke-WebRequest "https://github.com/nopSolutions/nopCommerce/releases/download/release-${env:NOP_VERSION}/nopCommerce_${env:NOP_VERSION}_Source.rar" -UseBasicParsing -OutFile "C:/nop-src.rar"; \
    Start-Process C:\7zip\7z.exe -ArgumentList 'x C:/nop-src.rar -oC:/nop' -Wait; \
    Remove-Item -Force "C:\7zsetup.exe"; \
    Remove-Item -Force "C:\nop-src.rar"; \
    Remove-Item -Force -Recurse "C:\7zip";

# Restore nuget packages
RUN $Env:PATH = "$Env:ProgramFiles\NuGet\;$Env:PATH;" \
    [Environment]::SetEnvironmentVariable('PATH', $Env:PATH, [EnvironmentVariableTarget]::Machine);

RUN nuget restore "C:/nop" -SolutionDirectory "C:/nop"

# Switch to cmd prompt instead of powershell because passing build options with spaces (e.g. Any Cpu) fails in powershell
SHELL ["cmd"]

# Build nopCommerce
RUN msbuild "C:/nop/nopCommerce.sln" /p:Configuration=Release /p:Platform="Any CPU" /p:Optimize=true /t:rebuild /verbosity:quiet

WORKDIR C:/nop

CMD ["cmd"]