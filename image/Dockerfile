# Designed to be run as 
# 
# docker run -it -p 9999:8888 ipython/latest

FROM ipython/scipystack

MAINTAINER IPython Project <ipython-dev@scipy.org>

# The ipython/ipython image has the full working copy of IPython
WORKDIR /srv/ipython/
RUN chmod a+rwX /srv/ipython/examples
ADD notebooks/ /srv/notebooks/

# Add Google Analytics templates
ADD ga/ /srv/ga/

# Dependencies for the example notebooks
RUN apt-get build-dep -y mpi4py && pip2 install scikit-image vincent dill networkx mpi4py && pip3 install scikit-image vincent dill networkx mpi4py

# Install R and the R kernel
RUN apt-get install -y r-base r-base-dev r-cran-rcurl
ADD R/ /srv/R/

# Julia Installation
RUN apt-get install software-properties-common python-software-properties -y && \
    add-apt-repository ppa:staticfloat/juliareleases && \
    add-apt-repository ppa:staticfloat/julia-deps && \
    apt-get update && \
    apt-get install julia -y && \
    apt-get install libnettle4

ADD Julia/ /srv/Julia/

EXPOSE 8888

RUN chmod a+rX /srv/R/ -R

# jupyter is our user
RUN useradd -m -s /bin/bash jupyter

USER jupyter
ENV HOME /home/jupyter
ENV SHELL /bin/bash
ENV USER jupyter

WORKDIR /home/jupyter/

# IJulia installation
RUN julia -e 'Pkg.add("IJulia")'
# Julia packages
RUN julia /srv/Julia/install_pkgs.jl

# Install kernel directly (for now)
RUN mkdir -p /home/jupyter/.ipython/kernels/julia/
RUN cp /srv/Julia/kernel.json /home/jupyter/.ipython/kernels/julia/kernel.json

# R installation
# Copying files instead of adding due to current permissions issue with ADD
RUN cp /srv/R/Renviron /home/jupyter/.Renviron
RUN cp /srv/R/Rprofile /home/jupyter/.Rprofile
RUN mkdir /home/jupyter/.R/

RUN cat /srv/R/install_kernel.R | R --no-save
RUN cat /srv/R/install_pkgs.R   | R --no-save

# Example notebooks 
RUN cp -r /srv/ipython/examples /home/jupyter/ipython_examples
RUN cp -r /srv/notebooks/* /home/jupyter/

RUN chown -R jupyter:jupyter /home/jupyter

RUN find . -name '*.ipynb' -exec ipython trust {} \;

CMD ipython3 notebook --no-browser --port 8888 --ip=0.0.0.0 --NotebookApp.base_url=/$RAND_BASE --NotebookApp.tornado_settings="{'template_path':['/srv/ga/', '/srv/ipython/IPython/html', '/srv/ipython/IPython/html/templates']}"
