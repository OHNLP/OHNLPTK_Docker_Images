# Basic setup
FROM ubuntu:latest
SHELL ["/bin/bash", "-c"]
# Set up apt, required GNU tools, and JDK
RUN apt-get update

RUN apt-get install -y wget curl unzip openjdk-17-jdk

ENV JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64

# Set up python/pip and deidentification dependencies
RUN apt-get install -y python3 python3-pip python3-venv python-is-python3
RUN python3 -m venv /default-python-env
RUN /default-python-env/bin/pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
RUN /default-python-env/bin/pip install transformers
RUN /default-python-env/bin/pip install ohnlptk-xlang-python==1.1.0 presidio-analyzer==2.2.33 presidio-anonymizer==2.2.33
RUN /default-python-env/bin/pip install charset_normalizer==3.4.0
RUN /default-python-env/bin/pip install spacy
RUN /default-python-env/bin/python -m spacy download en_core_web_sm

RUN exec bash \
    && source /default-python-env/bin/activate

# Set up OHNLPTK and Presidio Mdule
RUN wget \
    https://raw.githubusercontent.com/OHNLP/OHNLPTK_SETUP/refs/heads/master/install_or_update_ohnlptk.sh \
    && bash install_or_update_ohnlptk.sh

WORKDIR "/OHNLPTK"

RUN wget https://github.com/OHNLP/PresidioDeidentificationforOHNLPTK/releases/download/v1.0.2/PresidioDeidentificationForOHNLPTK.zip
RUN unzip PresidioDeidentificationForOHNLPTK.zip && rm PresidioDeidentificationForOHNLPTK.zip

WORKDIR "/OHNLPTK/python_resources"

RUN wget https://raw.githubusercontent.com/OHNLP/PresidioDeidentificationforOHNLPTK/refs/heads/master/src/build.py

RUN /default-python-env/bin/python build.py
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

# Cleanup
RUN rm -rf /var/lib/apt/lists/*
