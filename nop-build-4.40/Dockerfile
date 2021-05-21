FROM mcr.microsoft.com/dotnet/sdk:5.0-windowsservercore-ltsc2019

MAINTAINER dealdiane@netpotential.co.nz

ENV NOP_VERSION=4.40.3

# Download nopCommerce
RUN powershell -Command "\
	[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; \
    Invoke-WebRequest -OutFile C:\7zsetup.exe https://www.7-zip.org/a/7z1801-x64.exe; \
    Start-Process C:\7zsetup.exe -ArgumentList '/S /D=c:\7zip\' -Wait; \
    Invoke-WebRequest "https://github.com/nopSolutions/nopCommerce/releases/download/release-${env:NOP_VERSION}/nopCommerce_${env:NOP_VERSION}_Source.zip" -UseBasicParsing -OutFile "C:/nop-src.zip"; \
    Start-Process C:\7zip\7z.exe -ArgumentList 'x C:/nop-src.zip -oC:/nop-tmp' -Wait; \
    Move-Item -Force -Path "C:/nop-tmp/src" -Destination "C:/nop"; \
    Remove-Item "C:\nop-tmp" -Force -Recurse; \
    Remove-Item "C:\7zsetup.exe" -Force; \
    Remove-Item "C:\nop-src.zip" -Force; \
    Remove-Item "C:\7zip" -Force -Recurse;"

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

# Install node
ENV NODE_VERSION=15.10.0 \
    NODE_DOWNLOAD_SHA=2BF72DA4E5DDC485599F2ECA54AB7C59001D70BAEC7FF2BB050D9F4ED1B066A6

# From: https://github.com/aspnet/aspnet-docker/blob/master/1.1/nanoserver/sdk/Dockerfile
RUN powershell -Command "Invoke-WebRequest https://nodejs.org/dist/v${env:NODE_VERSION}/node-v${env:NODE_VERSION}-win-x64.zip -outfile node.zip; \
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

RUN powershell -Command "\
	Invoke-Expression ((New-Object Net.Webclient).DownloadString('https://chocolatey.org/install.ps1')); \
    &choco feature disable --name showDownloadProgress"

# Install package managers
RUN npm install -g gulp bower \
    choco install yarn -y

CMD ["cmd"]