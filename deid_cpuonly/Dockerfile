# Basic setup
FROM ubuntu:latest
ENV PATH="/root/miniconda3/bin:${PATH}"
ARG PATH="/root/miniconda3/bin:${PATH}"
SHELL ["/bin/bash", "-c"]
# Set up apt, required GNU tools, and JDK
RUN apt-get update

RUN apt-get install -y wget curl unzip openjdk-17-jdk && rm -rf /var/lib/apt/lists/*

ENV JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64

# Set up Anaconda
RUN wget \
    https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh \
    && mkdir /root/.conda \
    && bash Miniconda3-latest-Linux-x86_64.sh -b \
    && rm -f Miniconda3-latest-Linux-x86_64.sh

RUN exec bash \
    && conda init bash \
    && . /root/.bashrc \
    && conda activate

# Set up OHNLPTK and Presidio MOdule
RUN wget \
    https://raw.githubusercontent.com/OHNLP/OHNLPTK_SETUP/refs/heads/master/install_or_update_ohnlptk.sh \
    && bash install_or_update_ohnlptk.sh

WORKDIR "/OHNLPTK"

RUN wget https://github.com/OHNLP/PresidioDeidentificationforOHNLPTK/releases/download/v1.0.2/PresidioDeidentificationForOHNLPTK.zip
RUN unzip PresidioDeidentificationForOHNLPTK.zip && rm PresidioDeidentificationForOHNLPTK.zip

WORKDIR "/OHNLPTK/python_resources"

RUN wget https://raw.githubusercontent.com/OHNLP/PresidioDeidentificationforOHNLPTK/refs/heads/master/src/build.py
RUN conda install pytorch==2.5.1 torchvision==0.20.1 torchaudio==2.5.1 cpuonly -c pytorch
RUN conda install transformers
RUN python build.py
RUN rm build.py

# Get Demo Deidentification Config
WORKDIR "/OHNLPTK/configs"
RUN wget https://raw.githubusercontent.com/OHNLP/OHNLPTK_Docker_Images/refs/heads/main/deid_cpuonly/deid_config.json

# Create input/output folders and get demo data
WORKDIR "/OHNLPTK"
RUN mkdir input
RUN mkdir output

WORKDIR "/OHNLPTK/input"
RUN wget https://raw.githubusercontent.com/nlpie/nlptab-corpus/refs/heads/master/Documents/97_1677.txt && wget https://github.com/nlpie/nlptab-corpus/blob/master/Documents/100_2456.txt

# Setup Flink
WORKDIR "/OHNLPTK"
RUN wget https://archive.apache.org/dist/flink/flink-1.18.0/flink-1.18.0-bin-scala_2.12.tgz -O flink.tgz && tar -xf flink.tgz

# Prep for run
RUN chmod +x ./run_pipeline_local.sh && chmod +x ./run_backbone_configurator.sh && chmod +x ./package_modules_and_configs.sh
RUN java -cp bin/Plugin-Manager.jar org.ohnlp.backbone.pluginmanager.PluginManager Flink


