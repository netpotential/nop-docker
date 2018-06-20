FROM netpotential/msbuild:15.6

MAINTAINER dealdiane@netpotential.co.nz
    
RUN Invoke-WebRequest -OutFile C:\7zsetup.exe https://www.7-zip.org/a/7z1801-x64.exe; \
    Start-Process C:\7zsetup.exe -ArgumentList '/S /D=c:/7zip' -Wait; \
    Remove-Item C:\7zsetup.exe -Force
    
RUN $env:PATH = 'c:\7zip;' + $env:PATH; \
    [Environment]::SetEnvironmentVariable('PATH', $env:PATH, [EnvironmentVariableTarget]::Machine);
    
ENV NOP_VERSION 3.80

# Download nopCommerce
RUN [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; \
    Invoke-WebRequest "https://github.com/nopSolutions/nopCommerce/releases/download/release-${env:NOP_VERSION}/nopCommerce_${env:NOP_VERSION}_Source.rar" -UseBasicParsing -OutFile "C:/nop-src.rar"; \
    Start-Process C:\7zip\7z.exe -ArgumentList 'x C:/nop-src.rar -oC:/nop' -Wait; \
    Remove-Item "C:/nop-src.rar" -Force

# Restore nuget packages
RUN nuget restore "C:/nop" -SolutionDirectory "C:/nop"

# Switch to cmd prompt instead of powershell because passing build options with spaces (e.g. Any Cpu) fails in powershell
SHELL ["cmd"]

# Build nopCommerce
RUN msbuild "C:/nop/nopCommerce.sln" /p:Configuration=Release /p:Platform="Any CPU" /p:Optimize=true /t:rebuild /verbosity:quiet

WORKDIR C:/nop

CMD ["cmd"]