FROM mcr.microsoft.com/dotnet/framework/sdk:4.8-windowsservercore-ltsc2019

MAINTAINER dealdiane@netpotential.co.nz

# Install node
ENV NODE_VERSION=14.17.0 \
    NODE_DOWNLOAD_SHA=6582A7259C433E9F667DCC4ED3E5D68BC514CABA2EED40E4626C8B4C7E5ECD5C

# From: https://github.com/aspnet/aspnet-docker/blob/master/1.1/nanoserver/sdk/Dockerfile
RUN Invoke-WebRequest https://nodejs.org/dist/v${env:NODE_VERSION}/node-v${env:NODE_VERSION}-win-x64.zip -outfile node.zip; \
    if ((Get-FileHash node.zip -Algorithm sha256).Hash -ne $env:NODE_DOWNLOAD_SHA) { \
        Write-Host 'NODEJS CHECKSUM VERIFICATION FAILED!'; \
        exit 1; \
    }; \
    Expand-Archive node.zip -DestinationPath "C:/nodejs-tmp/"; \
    Move-Item "C:/nodejs-tmp/node-v${env:NODE_VERSION}-win-x64" -Destination "C:\nodejs"; \
    Remove-Item -Force "C:/nodejs-tmp/"; \
    Remove-Item -Force node.zip; \
    $env:PATH = 'C:\nodejs\;' + $env:PATH; \
    [Environment]::SetEnvironmentVariable('PATH', $env:PATH, [EnvironmentVariableTarget]::Machine);"

# Install chocolatey
ENV chocolateyUseWindowsCompression false

RUN [System.Net.ServicePointManager]::SecurityProtocol = 3072; \
    Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1')); \
    &choco feature disable --name showDownloadProgress"

# Install node build tools
# Normally should be npm install --global --production windows-build-tools but currently hangs and doesn't finish
ADD https://aka.ms/vs/15/release/vs_buildtools.exe C:\\vs15_buildtools.exe

RUN C:\\vs15_buildtools.exe --wait --quiet --norestart --nocache --add Microsoft.VisualStudio.Workload.VCTools --includeRecommended \
    Remove-Item -Force C:\vs15_buildtools.exe

# Install package managers
RUN npm install --global gulp bower \
    choco install yarn -y

# Compile sass
RUN npm rebuild node-sass

# Setup nopCommerce
ENV NOP_VERSION=4.40.3

# Download nopCommerce
RUN Invoke-WebRequest "https://github.com/nopSolutions/nopCommerce/releases/download/release-${env:NOP_VERSION}/nopCommerce_${env:NOP_VERSION}_Source.zip" -UseBasicParsing -OutFile 'C:/nop-src.zip';"
 
RUN Expand-Archive -LiteralPath 'C:\nop-src.zip' -DestinationPath c:\nop-tmp; \
    Move-Item -Force -Path 'C:/nop-tmp/src' -Destination 'C:/nop'; \
    Remove-Item 'C:\nop-tmp' -Force -Recurse; \
    Remove-Item 'C:\nop-src.zip' -Force;

# Based from https://github.com/nopSolutions/nopCommerce/blob/develop/Dockerfile
# Restore solution
WORKDIR /nop/
RUN dotnet restore NopCommerce.sln

# Build project   
WORKDIR /nop/Presentation/Nop.Web
RUN dotnet build Nop.Web.csproj -c Release

# Build plugins
WORKDIR /nop/Plugins/Nop.Plugin.DiscountRules.CustomerRoles
RUN dotnet build Nop.Plugin.DiscountRules.CustomerRoles.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.ExchangeRate.EcbExchange
RUN dotnet build Nop.Plugin.ExchangeRate.EcbExchange.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.ExternalAuth.Facebook
RUN dotnet build Nop.Plugin.ExternalAuth.Facebook.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Misc.Sendinblue
RUN dotnet build Nop.Plugin.Misc.Sendinblue.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Payments.CheckMoneyOrder
RUN dotnet build Nop.Plugin.Payments.CheckMoneyOrder.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Payments.Manual
RUN dotnet build Nop.Plugin.Payments.Manual.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Payments.PayPalSmartPaymentButtons
RUN dotnet build Nop.Plugin.Payments.PayPalSmartPaymentButtons.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Payments.PayPalStandard
RUN dotnet build Nop.Plugin.Payments.PayPalStandard.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Pickup.PickupInStore
RUN dotnet build Nop.Plugin.Pickup.PickupInStore.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Shipping.FixedByWeightByTotal
RUN dotnet build Nop.Plugin.Shipping.FixedByWeightByTotal.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Shipping.ShipStation
RUN dotnet build Nop.Plugin.Shipping.ShipStation.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Shipping.UPS
RUN dotnet build Nop.Plugin.Shipping.UPS.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Tax.Avalara
RUN dotnet build Nop.Plugin.Tax.Avalara.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Widgets.FacebookPixel
RUN dotnet build Nop.Plugin.Widgets.FacebookPixel.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Widgets.AccessiBe
RUN dotnet build Nop.Plugin.Widgets.AccessiBe.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Tax.FixedOrByCountryStateZip
RUN dotnet build Nop.Plugin.Tax.FixedOrByCountryStateZip.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Widgets.GoogleAnalytics
RUN dotnet build Nop.Plugin.Widgets.GoogleAnalytics.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Widgets.NivoSlider
RUN dotnet build Nop.Plugin.Widgets.NivoSlider.csproj -c Release

WORKDIR /nop/

CMD ["cmd"]