# Dockerfile to spin up a working Katana installation
FROM python:3.8

# Setup SSH Keys
RUN mkdir -p /root/.ssh
COPY ./deploy-key /root/.ssh/id_rsa
COPY ./deploy-key.pub /root/.ssh/id_rsa.pub
COPY ./ssh_config /root/.ssh/config
RUN chmod 600 /root/.ssh/id_rsa*
RUN chmod 600 /root/.ssh/config

# Install binary packages
RUN apt-get update && apt-get upgrade
# Base Dependencies
RUN apt-get install -y python-tk tk-dev libffi-dev libssl-dev pandoc \
	libgmp3-dev libzbar-dev tesseract-ocr xsel libpoppler-cpp-dev libmpc-dev \
	libdbus-glib-1-dev ruby libenchant-dev apktool nodejs groff binwalk \
	foremost tcpflow poppler-utils exiftool steghide stegsnow bison ffmpeg \
	libgd-dev
RUN apt-get install -y less

# Gem complained unless you bootstrapped rdoc first. I dont' know why.
RUN gem install rdoc --no-document
RUN gem install zsteg

# Compile and install npiet
RUN git clone https://github.com/gleitz/npiet /opt/npiet
# npiet includes `sys/malloc.h` which doesn't exist. Malloc definitions come
# from stdlib.h.
RUN sed -i 's|sys/malloc\.h|stdlib.h|g' /opt/npiet/npiet-foogol.y
RUN cd /opt/npiet && ./configure && make && make install

# Download jsteg
RUN wget -O /usr/local/bin/jsteg https://github.com/lukechampine/jsteg/releases/download/v0.3.0/jsteg-linux-amd64 && chmod +x /usr/local/bin/jsteg
RUN wget -O /usr/local/bin/slink https://github.com/lukechampine/jsteg/releases/download/v0.3.0/slink-linux-amd64 && chmod +x /usr/local/bin/slink

# Download, compile and install snow
RUN wget -O /usr/snow.tar.gz http://www.darkside.com.au/snow/snow-20130616.tar.gz
RUN cd /usr && tar -xvf snow.tar.gz
RUN cd /usr/snow-20130616/ && make 
RUN cp /usr/snow-20130616/snow /usr/local/bin/snow && chmod +x /usr/local/bin/snow

# Clone Katana Repository
RUN git clone --recursive git@gitlab.com:JohnHammond/katana.git /katana
# Install katana python dependencies
RUN cd /katana && pip install -r requirements.txt

# Create runtime data directory directory
RUN mkdir /data
WORKDIR /katana

# Run katana
ENTRYPOINT ["python", "-m", "katana"]
CMD ["-c", "/data/katana.ini", "-m", "monitor=/data/targets,outdir=/data/results"]