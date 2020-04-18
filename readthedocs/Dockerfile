FROM ubuntu:18.04
LABEL maintainer='\
Florian "floross" Rossiaud <o>, \
Simon "svvac" Wachter <himself@swordofpain.net>, \
Yevgeny Popovych <jmennius@gmail.com>'

ARG GITHUB_TOKEN

ENV DEBIAN_FRONTEND noninteractive
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8
ENV APPDIR=/app
ENV DJANGO_SETTINGS_MODULE=readthedocs.settings.container
#ENV PYTHON=/usr/bin/python3
ENV PYTHON=/venv/bin/python3
#ENV RTD_REF=3.11.6
ENV RTD_REF=4.0.3
#ENV READTHEDOCS 1

# Set locale to UTF-8
RUN apt-get -qq update && \
    apt-get install -y apt-utils && \ 
    echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    apt-get install -y locales

# Install basic dependencies
RUN apt-get -qq update && \
    apt-get install -qq --no-install-recommends \
        python-setuptools build-essential python-dev python-pip git libevent-dev \
        python3 python3-virtualenv python3-pip python3-dev \
        python3.6 python3.6-dev python3.6-venv python3-distutils \
        libpq-dev postgresql-client \
        libxml2-dev libxslt1-dev gettext texlive texlive-latex-extra \
        wget \
        nginx

RUN update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.6 1

# Setting conda
RUN wget -c https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
chmod +x Miniconda3-latest-Linux-x86_64.sh && \
bash ./Miniconda3-latest-Linux-x86_64.sh -b -f -p /usr/local

#RUN ln -s /usr/bin/python3 /usr/local/bin/python3.7

RUN python3 -m pip install --upgrade pip
RUN python3 -m pip install setuptools

# Install test dependencies
RUN python3 -m pip install wheel && \
    python3 -m install virtualenv && \
    python3 -m pip install supervisor

RUN python3 -V

# Setting up virtualenv
RUN python3 -m venv venv && \
    . venv/bin/activate

RUN ${PYTHON} -V
RUN ${PYTHON} -m pip install virtualenv
RUN pip install virtualenv


# Install test dependencies
RUN $PYTHON -m pip install wheel && \
    $PYTHON -m pip install supervisor

# Add user py
RUN adduser --gecos 'py' --disabled-password py

RUN mkdir -p $APPDIR
WORKDIR $APPDIR

# Setup read the docs

## Extract readthedocs
RUN wget -q --no-check-certificate -O - \
        https://github.com/rtfd/readthedocs.org/archive/${RTD_REF}.tar.gz | \
        tar -xzf - --strip-components 1 && \
    $PYTHON -m pip install -U -r $APPDIR/requirements/deploy.txt && \
    #$PYTHON -m pip install -U GitPython>=3.0.8 && \
    #$PYTHON -m pip install -U gitdb2>=3.0.2 && \
    #$PYTHON -m pip install -U celery>=4.3 && \
    $PYTHON -m pip install 'git+https://github.com/rtfd/readthedocs-sphinx-ext.git'

RUN if [ -n "$GITHUB_TOKEN" ] ; \
        then \
        git clone --depth 1 https://${GITHUB_TOKEN}@github.com/readthedocs/readthedocs-ext ; \
        $PYTHON -m pip install --no-cache-dir -e readthedocs-ext ; \
        fi

## Copy configuration, scripts and patch RTD
COPY . /
RUN find /patches -type f -exec patch -p1 -i {} \; && rm -rf /patches

## Copy special configuration for read the docs
RUN ln -s $APPDIR/manage.py $APPDIR/readthedocs/manage.py && \
    ln -s /etc/nginx/sites-available/readthedocs /etc/nginx/sites-enabled/readthedocs && \
    rm /etc/nginx/sites-enabled/default && \
    chmod +x $APPDIR/bin/* && \
    chown -R py .


# Docker config
EXPOSE 80
VOLUME [ "$APPDIR/user_builds" ]

CMD [ "supervisord", "-c", "/etc/supervisord.conf", "-n" ]
