# A Docker container to run the OHDSI/Achilles analysis tool
FROM ubuntu:focal

LABEL maintainer="Taha Abdul-Basser <ta2471@cumc.columbia.edu>"

# Install java, R and required packages and clean up.
RUN apt-get update && \
    apt-get install --no-install-recommends -y software-properties-common && \
    add-apt-repository ppa:marutter/rrutter4.0 && \
    add-apt-repository ppa:c2d4u.team/c2d4u4.0+ && \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --allow-unauthenticated \
    r-base \
    r-cran-devtools \
    r-cran-httr \
    r-cran-rjson \
    r-cran-stringr \
    r-cran-rjava \
    r-cran-dbi \
    r-cran-ffbase \
    r-cran-urltools \
    libxml2-dev \
    littler \
    locales \
    openjdk-8-jdk \
    && rm -rf /var/lib/apt/lists/* \
    && R CMD javareconf

# Set default locale
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
    && locale-gen en_US.utf8 \
    && /usr/sbin/update-locale LANG=en_US.UTF-8

ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8

# Install OHDSI/ParallelLogger
RUN R -e "install.packages( \
    c( \
    'XML', \
    'RJSONIO' \
    ), \
    repos='http://cran.rstudio.com/', \
    ) "

# Install Achilles requirements that need to be installed from source
RUN echo 'options(repos=structure(c(CRAN="https://cloud.r-project.org/")))' > /root/.Rprofile && \
    /usr/share/doc/littler/examples/install.r remotes && \
    /usr/share/doc/littler/examples/install.r docopt && \
    /usr/share/doc/littler/examples/install.r openxlsx && \
    /usr/share/doc/littler/examples/install.r httr && \
    /usr/share/doc/littler/examples/install.r rjson && \
    /usr/share/doc/littler/examples/install.r R.oo && \
    /usr/share/doc/littler/examples/install.r formatR && \
    /usr/share/doc/littler/examples/install.r R.utils && \
    /usr/share/doc/littler/examples/install.r snow && \
    /usr/share/doc/littler/examples/install.r mailR && \
    /usr/share/doc/littler/examples/install.r dplyr && \
    /usr/share/doc/littler/examples/install.r readr && \
    /usr/share/doc/littler/examples/installGithub.r \
    OHDSI/SqlRender \
    OHDSI/DatabaseConnectorJars \
    OHDSI/DatabaseConnector \
    OHDSI/ParallelLogger \
    && rm -rf /tmp/downloaded_packages/ /tmp/*.rds

# Configure workspace
WORKDIR /opt/app
ENV PATH /opt/app:$PATH
VOLUME /opt/app/output

# Add project files to container
# v1.6.3
ARG ACHILLES_VERSION=master
RUN git clone --depth=1 -b ${ACHILLES_VERSION} https://github.com/OHDSI/Achilles.git /opt/app
COPY . /opt/app/
RUN ls -lisa

# Install Achilles from source
RUN R CMD INSTALL /opt/app \
    && rm -rf /tmp/downloaded_packages/ /tmp/*.rds \
    && find /opt/app -mindepth 1 -not \( -wholename /opt/app/docker-run -or -wholename /opt/app/output \) -delete

# Create a non-root user with full access to the /opt/app directory
RUN groupadd -g 10001 achilles && \
    useradd -u 10001 -g achilles achilles && \
    mkdir /opt/app/output && \
    chown -R achilles /opt/app/

USER 10001
VOLUME /opt/app/output

# Define run script as default command
CMD ["docker-run.R"]
