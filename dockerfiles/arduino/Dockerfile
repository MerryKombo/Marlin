# This Dockerfile is used to create a Jenkins SSH agent with Arduino development environment.

# Use jenkins/ssh-agent:5.38.0 as the base image
FROM jenkins/ssh-agent:6.7.0 AS ssh-agent

# Install necessary packages including arduino-mk, build-essential, ca-certificates, curl, git, python3 and its related packages
# This is to ensure that the necessary dependencies for Arduino development and Jenkins agent are installed.
RUN apt-get update && apt-get install -y --no-install-recommends arduino-mk \
    build-essential \
    ca-certificates \
    curl \
    git \
    python3 \
    python3-pip \
    python-is-python3 \
    python3-venv \
    python3-setuptools \
    python3-wheel \
    python3-dev \
    && apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Set SHELL flags for RUN commands to allow -e and pipefail
# This is to ensure that the Docker build fails if any command fails.
SHELL ["/bin/bash", "-eo", "pipefail", "-c"]

# Create a Python virtual environment in /opt/venv.
# This is to isolate the Python environment for this Docker image.
RUN python3 -m venv /opt/venv

# Activate the virtual environment by adding its bin directory to the PATH.
# This is to ensure that the Python packages installed in the next step are installed in the virtual environment.
ENV PATH="/opt/venv/bin:$PATH"

# Install required Python packages in the virtual environment.
# These packages are required for Arduino development and Jenkins agent.
RUN pip install pyserial docker-py feedparser nosexcover prometheus_client pycobertura pylint pytest pytest-cov requests setuptools sphinx pyinstaller wheel

# Add the PATH environment variable to /etc/environment so that it is available to all users and processes.
# This is to ensure that the Python virtual environment is available to all users and processes.
RUN echo "PATH=${PATH}" >> /etc/environment

# Install Arduino CLI
# This is to provide a command-line interface for Arduino development.
RUN curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | sh

# Add /usr/local/bin to PATH
# This is to ensure that the Arduino CLI and other tools installed in /usr/local/bin are available in the PATH.
ENV PATH="/home/jenkins/bin/:/usr/local/bin:${PATH}"

# Add the PATH environment variable to /etc/environment and change the owner of the JENKINS_AGENT_HOME directory to jenkins
# This is to ensure that the PATH environment variable is available to all users and processes, and that the Jenkins agent has the necessary permissions.
RUN echo "PATH=${PATH}" >> /etc/environment && chown -R jenkins:jenkins "${JENKINS_AGENT_HOME}"

# Add jenkins to the dialout group, clone the arduino-esp32 repository and run the get.py script
# This is to ensure that the Jenkins agent has the necessary permissions to access the serial port, and to install the ESP32 core for Arduino.
RUN usermod -a -G dialout jenkins && \
    mkdir -p /home/jenkins/Arduino/hardware/espressif && \
    cd /home/jenkins/Arduino/hardware/espressif && \
    git clone https://github.com/espressif/arduino-esp32.git esp32 && \
    cd esp32/tools && \
    python3 get.py

# Clone the Adafruit NeoPixel library and other Adafruit libraries
# These libraries are required for Arduino development.
RUN mkdir /home/jenkins/Arduino/libraries
RUN git clone https://github.com/adafruit/Adafruit_NeoPixel.git ~/Arduino/libraries/Adafruit_NeoPixel
RUN git clone https://github.com/adafruit/Adafruit_SSD1306.git ~/Arduino/libraries/Adafruit_SSD1306
RUN git clone https://github.com/adafruit/Adafruit-GFX-Library.git ~/Arduino/libraries/Adafruit_GFX
RUN git clone https://github.com/adafruit/Adafruit_BusIO.git ~/Arduino/libraries/Adafruit_BusIO

# Verify Arduino CLI installation
# This is to ensure that the Arduino CLI is installed correctly.
RUN arduino-cli version

# Copy the Arduino CLI configuration file to the home directory of the Jenkins agent.
COPY .cli-config.yml ./.cli-config.yml

# Install the ESP8266 core and several libraries using the Arduino CLI.
# These cores and libraries are required for Arduino development.
RUN arduino-cli core install esp8266:esp8266 --config-file ./.cli-config.yml && \
        arduino-cli lib install "Adafruit ADS1X15" && arduino-cli lib install "OneWire" && \
        arduino-cli lib install "DallasTemperature" && arduino-cli lib install "NTPClientLib" && \
        arduino-cli lib install "Time" && arduino-cli lib install "NtpClient" && \
        arduino-cli lib install "WiFiManager" && arduino-cli lib install "RemoteDebug" && \
        arduino-cli lib install "ArduinoJson"
# For the sake of rebuilding the image.
