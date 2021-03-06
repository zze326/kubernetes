# Copyright 2020 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM mcr.microsoft.com/windows/servercore:ltsc2019 as prep

ENV PS_VERSION=6.2.7
ADD https://github.com/PowerShell/PowerShell/releases/download/v$PS_VERSION/PowerShell-$PS_VERSION-win-x64.zip /PowerShell/powershell.zip

RUN cd C:\PowerShell &\
    tar.exe -xf powershell.zip &\
    del powershell.zip &\
    mklink powershell.exe pwsh.exe

FROM mcr.microsoft.com/windows/nanoserver:1809

COPY --from=prep /PowerShell /PowerShell

# set a fixed location for the Module analysis cache
ENV LOCALAPPDATA="C:\Users\ContainerAdministrator\AppData\Local" \
    PSModuleAnalysisCachePath="C:\Users\ContainerAdministrator\AppData\Local\Microsoft\Windows\PowerShell\docker\ModuleAnalysisCache" \
    # Persist %PSCORE% ENV variable for user convenience
    PSCORE="C:\PowerShell\pwsh.exe"

# use downloaded powershell
USER ContainerAdministrator
RUN setx /M PATH "C:\Powershell\;%PATH%"

# intialize powershell module cache
RUN powershell \
        -NoLogo \
        -NoProfile \
        -Command " \
          $stopTime = (get-date).AddMinutes(15); \
          $ErrorActionPreference = 'Stop' ; \
          $ProgressPreference = 'SilentlyContinue' ; \
          while(!(Test-Path -Path $env:PSModuleAnalysisCachePath)) {  \
            Write-Host "'Waiting for $env:PSModuleAnalysisCachePath'" ; \
            if((get-date) -gt $stopTime) { throw 'timout expired'} \
            Start-Sleep -Seconds 6 ; \
          }"
