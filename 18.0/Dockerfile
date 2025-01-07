FROM python:3.12-slim-bullseye

# Install system dependencies
RUN apt-get update && apt-get install -y \
    curl \
    dirmngr \
    fonts-noto-cjk \
    gnupg \
    libldap2-dev \
    libpq-dev \
    libsasl2-dev \
    libssl-dev \
    libxml2-dev \
    libxslt1-dev \
    git \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Add PostgreSQL repository
RUN curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg \
    && echo "deb http://apt.postgresql.org/pub/repos/apt bullseye-pgdg main" > /etc/apt/sources.list.d/pgdg.list \
    && apt-get update \
    && apt-get install -y postgresql-client-14 \
    && rm -rf /var/lib/apt/lists/*

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
ADD https://raw.githubusercontent.com/odoo/odoo/18.0/requirements.txt /
COPY requirementslocal.txt /

# Upgrade pip and install build dependencies
RUN pip install --upgrade pip && \
    pip install wheel cython

# Install the rest of requirements
RUN pip install --no-cache-dir -r /requirementslocal.txt || true
RUN pip install --no-cache-dir -r /requirements.txt || true

# Install Node.js, npm and rtlcss
RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g rtlcss \
    && rm -rf /var/lib/apt/lists/*

# Create odoo user
RUN useradd -ms /bin/bash odoo

# Set permissions and Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN mkdir -p /mnt/extra-addons /var/lib/odoo/sessions \
&& chown -R odoo /mnt/extra-addons /var/lib/odoo

# Expose Odoo services
EXPOSE 8069 8071 8072

ENV ODOO_RC=/etc/odoo/odoo.conf

COPY wait-for-psql.py /usr/local/bin/wait-for-psql.py
# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo-bin"]
