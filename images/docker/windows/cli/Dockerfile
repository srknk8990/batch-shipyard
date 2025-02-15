# Dockerfile for Azure/batch-shipyard CLI (Windows)
# Adapted from: https://github.com/StefanScherer/dockerfiles-windows/blob/master/python/Dockerfile

FROM python:3.7.4-windowsservercore-ltsc2016
MAINTAINER Fred Park <https://github.com/Azure/batch-shipyard>

ENV chocolateyUseWindowsCompression false
RUN [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 ; \
    iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1')); \
    choco install --no-progress -y git -params "/GitAndUnixToolsOnPath"

ARG GIT_BRANCH
ARG GIT_COMMIT

WORKDIR C:\\batch-shipyard
RUN git clone -b $Env:GIT_BRANCH --single-branch https://github.com/Azure/batch-shipyard.git C:\batch-shipyard ; \
    git checkout $Env:GIT_COMMIT ; \
    pip install --no-cache-dir -r requirements.txt ; \
    pip install --no-cache-dir --no-deps -r req_nodeps.txt ; \
    Remove-Item .git -Force -Recurse ; \
    Remove-Item .git* -Force -Recurse ; \
    Remove-Item .vsts -Force -Recurse ; \
    Remove-Item .travis.yml -Force -Recurse ; \
    Remove-Item *.yml -Force -Recurse ; \
    Remove-Item install* -Force -Recurse

RUN python -m compileall C:\Python\Lib\site-packages ; \
    python -m compileall C:\batch-shipyard\shipyard.py ; \
    python -m compileall C:\batch-shipyard\convoy ; \
	exit 0

FROM mcr.microsoft.com/windows/nanoserver:sac2016

COPY --from=0 /Python /Python
COPY --from=0 /batch-shipyard /batch-shipyard

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

ENV PYTHON_VERSION 3.7.4
ENV PYTHON_PIP_VERSION 18.1

RUN $env:PATH = 'C:\Python;C:\Python\Scripts;{0}' -f $env:PATH ; \
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Environment\' -Name Path -Value $env:PATH ; \
    mkdir $env:APPDATA\Python\Python37\site-packages ; \
    Invoke-WebRequest 'https://bootstrap.pypa.io/get-pip.py' -OutFile 'get-pip.py' -UseBasicParsing ; \
    $replace = ('import tempfile{0}import site{0}site.getusersitepackages()' -f [char][int]10) ; \
    Get-Content get-pip.py | Foreach-Object { $_ -replace 'import tempfile', $replace } | Out-File -Encoding Ascii getpip.py ; \
    $pipInstall = ('pip=={0}' -f $env:PYTHON_PIP_VERSION) ; \
    python getpip.py $pipInstall ; \
    Remove-Item get-pip.py ; \
    Remove-Item getpip.py

WORKDIR C:\\batch-shipyard
ENTRYPOINT ["python", "shipyard.py"]
