FROM python:2.7.11
MAINTAINER Hibou Corp. <hello@hibou.io>

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN set -x; \
        apt-get update \
        && apt-get install -y --no-install-recommends \
            ca-certificates \
            curl \
            nano \
            htop \
            python-pyinotify \
            python-renderpm \
            python-support \
            python-watchdog \
            nodejs \
            npm \
        && curl -o wkhtmltox.deb -SL http://nightly.odoo.com/extra/wkhtmltox-0.12.1.2_linux-jessie-amd64.deb \
        && echo '40e8b906de658a2221b15e4e8cd82565a47d7ee8 wkhtmltox.deb' | sha1sum -c - \
        && dpkg --force-depends -i wkhtmltox.deb \
        && apt-get -y install -f --no-install-recommends \
        && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

RUN pip install psycogreen

RUN npm install -g less \
    && npm install -g less-plugin-clean-css \
    && ln -s `which nodejs` /bin/node \
    && ln -s `which lessc` /bin/lessc

# Install Odoo
ENV ODOO_VERSION 8.0
ENV ODOO_RELEASE 20160428
RUN set -x; \
        curl -o odoo.deb -SL http://nightly.odoo.com/${ODOO_VERSION}/nightly/deb/odoo_${ODOO_VERSION}.${ODOO_RELEASE}_all.deb \
        && echo 'f5109ed62235380304e8b953c60f5c0fa9c5c5cd odoo.deb' | sha1sum -c - \
        && dpkg --force-depends -i odoo.deb \
        && apt-get update \
        && apt-get -y install -f --no-install-recommends \
        && rm -rf /var/lib/apt/lists/* odoo.deb

RUN apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false npm

# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./openerp-server.conf /etc/odoo/
RUN chown odoo /etc/odoo/openerp-server.conf

# Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN mkdir -p /mnt/extra-addons \
        && chown -R odoo /mnt/extra-addons
VOLUME ["/var/lib/odoo", "/mnt/extra-addons"]

# Expose Odoo services
EXPOSE 8069 8071

# Set the default config file
ENV OPENERP_SERVER /etc/odoo/openerp-server.conf

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["openerp-server"]
