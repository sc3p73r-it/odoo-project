FROM odoo:17.0

USER root

# (Optional) Install additional packages
RUN apt-get update && apt-get install -y \
    nano \
    net-tools \
    && rm -rf /var/lib/apt/lists/*

USER odoo
