# Use an official Python runtime as a parent image
FROM python:3.10-slim-buster

ARG ARCH

# Set the working directory in the container to /app
WORKDIR /

# Install Debian dependencies
RUN apt-get update && apt-get install -y \
    curl \
    dirmngr \
    fonts-noto-cjk \
    gnupg \
    libssl-dev \
    node-less \
    npm \
    python3-renderpm \
    python3-watchdog \
    xz-utils \
    libpq-dev \
    libldap2-dev \
    libsasl2-dev \
    libffi-dev \
    git \
    build-essential \
    libxml2-dev \
    libxslt-dev \
    zlib1g-dev \
    fontconfig \
    libxrender1 \
    xfonts-75dpi \
    xfonts-base \
    libxext6 \
    libx11-6 \
    wget \
    gdebi \
    libjpeg-dev \
    libfreetype6-dev \
    liblcms2-dev \
    libblas-dev \
    liblapack-dev \
    gfortran \
    libatlas-base-dev \
    python3-dev \
    libbz2-dev \
    libjpeg62-turbo-dev \
    postgresql-client

RUN curl -o wkhtmltox.deb -SL https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.buster_amd64.deb
COPY wkhtmltopdf /usr/local/bin/wkhtmltopdf_arm
RUN ln -s /usr/local/bin/wkhtmltopdf_arm /usr/local/bin/wkhtmltopdf

RUN if [ "$ARCH" = "amd64" ]; then \
    dpkg -i wkhtmltox.deb && \
    apt-get -y install -f --no-install-recommends && \
    rm -rf wkhtmltox.deb /var/lib/apt/lists/*; \
fi
#    else \
#    ln -s /usr/local/bin/wkhtmltopdf_arm /usr/local/bin/wkhtmltopdf; \
# fi

# Copy files to root directory
COPY entrypoint.sh /
COPY odoo.conf /etc/odoo/
# COPY requirements.txt /
ADD https://raw.githubusercontent.com/odoo/odoo/17.0/requirements.txt /
COPY requirementslocal.txt /

RUN pip install --upgrade pip

# Install dependencies from the local requirements.txt
RUN pip install --no-cache-dir -r /requirements.txt
RUN pip install --no-cache-dir -r /requirementslocal.txt

# Install IPython
RUN pip install ipython==7.31.1

# Install rtlcss (on Debian buster)
RUN npm install -g rtlcss

# Install additional dependencies
RUN pip install pdf417gen
RUN pip install google_auth
RUN pip install numpy

# Create odoo user
RUN useradd -ms /bin/bash odoo

# Set permissions and Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN mkdir -p /mnt/extra-addons /var/lib/odoo/sessions \
&& chown -R odoo /mnt/extra-addons /var/lib/odoo

# Expose Odoo services
EXPOSE 8069 8071 8072

ENV ODOO_RC /etc/odoo/odoo.conf

COPY wait-for-psql.py /usr/local/bin/wait-for-psql.py
# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo-bin"]