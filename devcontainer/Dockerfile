# Building the image:
#
# 1. Be sure to have C/C++test Standard (optionally also C/C++test CT) distribution files (.tar.gz) in the same directory as this Dockerfile.
# 2. Provide an OpenAI API key and a license server URL as build arguments.
#    IMPORTANT: The OpenAI API key will be stored in the image in plain text,
#    so make sure to use a key with limited permissions and do not share the image with untrusted parties.
# 3. Run the following command in this directory to build the image:
#    > docker build --build-arg OPENAI_API_KEY=YOURKEY --build-arg LICENSE_SERVER_URL=https://ls.company.com:8443 -t cpptest:latest .
# 4. Verify the image was created successfully:
#    > docker images cpptest:latest
#
# Using the image:
#
# A. Run the container directly:
#    > docker run -it --rm cpptest:latest bash
# B. Use the image with the VS Code Remote Containers extension by creating a '.devcontainer/devcontainer.json' file in your project with the following content:
# {
#     "image": "cpptest:latest",
#     "runArgs": [ "--rm" ]
# }
# C. Use the pre-installed GitHub Actions runner:
#    Run the container:
#    > docker run --rm -it cpptest:latest bash
#    Activate / register / run GitHub Runner inside the container (following instructions from GitHub):
#    > cd actions-runner
#    > ./config.sh --url https://github.com/my/project --token YOURTOKENFROMGITHUB
#    > ./run.sh
#

# Base image
FROM debian:13

# build-time parameters
ARG OPENAI_API_KEY=""
ARG LICENSE_SERVER_URL=""

# Install toolchains and utilities
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        gcc g++ make cmake git mc vim wget curl fontconfig sudo openssh-client ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# Install python, node and jre
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        python3.13 python-is-python3 nodejs npm openjdk-21-jre-headless unzip && \
    rm -rf /var/lib/apt/lists/*

# Configure default user
RUN groupadd parasoft && \
    useradd -d /home/parasoft -m -g parasoft -G sudo -s /bin/bash parasoft && \
    echo "parasoft:parasoft" | chpasswd

# Install C/C++test Standard and/or CT - be sure to have the original cpptest distribution files (.tar.gz)
WORKDIR /opt/parasoft
COPY parasoft_cpptest_*.tar.gz .
RUN \
    for cpptest_distro in parasoft_cpptest_*.tar.gz; do tar -zxf "$cpptest_distro"; done && \
    rm -f parasoft_cpptest_*.tar.gz

# Pre-install the VS Code extension from the C/C++test distribution, if available
WORKDIR /home/parasoft
RUN \
    if [ -f /opt/parasoft/cpptest/integration/vscode/*.vsix ] ; then \
        mkdir -p .vscode-server/extensions && \
        cp /opt/parasoft/cpptest/integration/vscode/*.vsix .vscode-server/extensions/ && \
        cd .vscode-server/extensions && \
        ext_version=$(ls *.vsix | grep -oP '\d+\.\d+\.\d+') && \
        unzip *.vsix && \
        mv extension parasoft.vscode-cpptest-$ext_version && \
        mv extension.vsixmanifest parasoft.vscode-cpptest-$ext_version/.vsixmanifest && \
        rm *.vsix && rm *.xml && \
        cd ../.. && \
        mkdir -p .vscode-server/data/Machine && \
        echo '{ "cpptest.installationLocation": "/opt/parasoft/cpptest" }' > .vscode-server/data/Machine/settings.json \
    ; else \
        echo "Skipped pre-installing Visual Studio Code extension" \
    ; fi

# Pre-install GitHub Actions Runner
WORKDIR /home/parasoft
RUN \
    mkdir actions-runner && cd actions-runner && \
    curl -o gh_runner_latest.json -L https://api.github.com/repos/actions/runner/releases/latest && \
    gh_runner_version=`grep tag_name gh_runner_latest.json | cut -d'"' -f4 | sed 's/^v//'` && \
    gh_runner_arch=$(dpkg --print-architecture) && \
    curl -o actions-runner-latest.tar.gz -L https://github.com/actions/runner/releases/download/v$gh_runner_version/actions-runner-linux-$gh_runner_arch-$gh_runner_version.tar.gz && \
    tar xzf ./actions-runner-latest.tar.gz && \
    ./bin/installdependencies.sh

# Pre-install GitHub CLI
WORKDIR /home/parasoft
RUN \
    (type -p wget >/dev/null || (sudo apt update && sudo apt install wget -y)) \
	&& sudo mkdir -p -m 755 /etc/apt/keyrings \
	&& out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
	&& cat $out | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
	&& sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
	&& sudo mkdir -p -m 755 /etc/apt/sources.list.d \
	&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
	&& sudo apt update \
	&& sudo apt install gh -y \
    && rm -rf /var/lib/apt/lists/*

# Pre-install Codex - be sure to provide an OpenAI API key as a build argument!
WORKDIR /home/parasoft
RUN \
    npm i -g @openai/codex && \
    mkdir -p .codex && \
    echo 'model = "gpt-5.3-codex"' >> .codex/config.toml && \
    echo 'model_reasoning_effort = "medium"' >> .codex/config.toml && \
    echo '[projects."/home/parasoft"]' >> .codex/config.toml && \
    echo 'trust_level = "trusted"' >> .codex/config.toml && \
    echo '{"auth_mode":"apikey","OPENAI_API_KEY":"'$OPENAI_API_KEY'"}' > .codex/auth.json

# Create C/C++test configuration files - be sure to provide a license server URL as a build argument!
WORKDIR /home/parasoft
RUN \
# Pre-configure C/C++test Standard
    echo 'parasoft.eula.accepted=true' >> cpptestcli.properties && \
    echo 'cpptest.license.use_network=true' >> cpptestcli.properties && \
    echo 'cpptest.license.network.edition=custom_edition' >> cpptestcli.properties && \
    echo 'cpptest.license.custom_edition_features=C++test,Static Analysis,Flow Analysis,Unit Test,Coverage,Automation,DTP Publish,Desktop Command Line,AUTOSAR Rules,CWE Rules,DISA STIG Rules,HIC++ Rules,JSF Rules,MISRA Rules,MISRA C 2012 Rules,MISRA C++ 2023 Rules,OWASP Rules,Security Rules,SEI CERT C Rules,SEI CERT C++ Rules,Code Dependency Export,LLM Integration' >> cpptestcli.properties && \
    echo 'license.network.use.specified.server=true' >> cpptestcli.properties && \
    echo 'license.network.url='$LICENSE_SERVER_URL >> cpptestcli.properties && \
    echo 'license.network.auth.enabled=false' >> cpptestcli.properties && \
# Pre-configure C/C++test CT
    echo 'parasoft.eula.accepted=true' >> cpptestct.properties && \
    echo 'cpptestct.license.use_network=true' >> cpptestct.properties && \
    echo 'cpptestct.license.network.edition=compliance_edition' >> cpptestct.properties && \
    echo 'license.network.use.specified.server=true' >> cpptestct.properties && \
    echo 'license.network.url='$LICENSE_SERVER_URL >> cpptestct.properties && \
    echo 'license.network.auth.enabled=false' >> cpptestct.properties

# Update permissions after installing as root
RUN \
    chown -R parasoft:parasoft /opt/parasoft && \
    chown -R parasoft:parasoft /home/parasoft

# Configure environment
ENV PATH="/opt/parasoft/cpptest-ct:/opt/parasoft/cpptest-ct/bin:/opt/parasoft/cpptest:/opt/parasoft/cpptest/bin:$PATH" CPPTEST_HOME="/opt/parasoft/cpptest-ct"
WORKDIR /home/parasoft
USER parasoft:parasoft
