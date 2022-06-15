FROM mcr.microsoft.com/dotnet/sdk:6.0

MAINTAINER dealdiane@netpotential.co.nz

# Install node
ENV NODE_VERSION=14.17.0 \
    NODE_DOWNLOAD_SHA=6582A7259C433E9F667DCC4ED3E5D68BC514CABA2EED40E4626C8B4C7E5ECD5C \
    NOP_VERSION=4.50.2 \
    VS_BUILDTOOLS_URI=https://aka.ms/vs/15/release/vs_buildtools.exe

USER ContainerAdministrator

SHELL ["pwsh", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'Continue'; $verbosePreference='Continue';"]

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

# Install node build tools
# Normally should be npm install --global --production windows-build-tools but currently hangs and doesn't finish
# Install VS Build Tools
RUN Invoke-WebRequest $env:VS_BUILDTOOLS_URI -outfile vs_BuildTools.exe; \
    .\vs_BuildTools.exe --add Microsoft.VisualStudio.Workload.VCTools --quiet --norestart --nocache --wait --includeRecommended; \
    if ($err = dir $Env:TEMP -Filter dd_setup_*_errors.log | where Length -gt 0 | Get-Content) { throw $err }; \
    Remove-Item vs_BuildTools.exe

# Install package managers
RUN npm install --global gulp bower yarn

# Compile sass
RUN npm rebuild node-sass

# Download nopCommerce
RUN Invoke-WebRequest "https://github.com/nopSolutions/nopCommerce/releases/download/release-${env:NOP_VERSION}/nopCommerce_${env:NOP_VERSION}_Source.zip" -UseBasicParsing -OutFile 'C:/nop-src.zip'; \
    Expand-Archive -LiteralPath 'C:\nop-src.zip' -DestinationPath c:\nop-tmp; \
    if (Test-Path -Path 'c:/nop-tmp/src') { \
        Move-Item -Force -Path 'C:/nop-tmp/src' -Destination 'C:/nop'; \
        Remove-Item 'C:\nop-tmp' -Force -Recurse; \
    } else { \
        Move-Item -Force -Path 'C:/nop-tmp' -Destination 'C:/nop'; \
    } \
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
WORKDIR /nop/Plugins/Nop.Plugin.Misc.WebApi.Frontend
RUN dotnet build Nop.Plugin.Misc.WebApi.Frontend.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.MultiFactorAuth.GoogleAuthenticator
RUN dotnet build Nop.Plugin.MultiFactorAuth.GoogleAuthenticator.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Payments.CheckMoneyOrder
RUN dotnet build Nop.Plugin.Payments.CheckMoneyOrder.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Payments.Manual
RUN dotnet build Nop.Plugin.Payments.Manual.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Payments.PayPalCommerce
RUN dotnet build Nop.Plugin.Payments.PayPalCommerce.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Payments.PayPalStandard
RUN dotnet build Nop.Plugin.Payments.PayPalStandard.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Pickup.PickupInStore
RUN dotnet build Nop.Plugin.Pickup.PickupInStore.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Shipping.EasyPost
RUN dotnet build Nop.Plugin.Shipping.EasyPost.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Shipping.FixedByWeightByTotal
RUN dotnet build Nop.Plugin.Shipping.FixedByWeightByTotal.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Shipping.UPS
RUN dotnet build Nop.Plugin.Shipping.UPS.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Tax.Avalara
RUN dotnet build Nop.Plugin.Tax.Avalara.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Tax.FixedOrByCountryStateZip
RUN dotnet build Nop.Plugin.Tax.FixedOrByCountryStateZip.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Widgets.AccessiBe
RUN dotnet build Nop.Plugin.Widgets.AccessiBe.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Widgets.FacebookPixel
RUN dotnet build Nop.Plugin.Widgets.FacebookPixel.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Widgets.GoogleAnalytics
RUN dotnet build Nop.Plugin.Widgets.GoogleAnalytics.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Widgets.NivoSlider
RUN dotnet build Nop.Plugin.Widgets.NivoSlider.csproj -c Release
WORKDIR /nop/Plugins/Nop.Plugin.Widgets.What3words
RUN dotnet build Nop.Plugin.Widgets.What3words.csproj -c Release

WORKDIR /nop/

CMD ["cmd"]