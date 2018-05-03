# This Source Code Form is subject to the terms of the Mozilla Public
# License, v. 2.0. If a copy of the MPL was not distributed with this
# file, You can obtain one at http://mozilla.org/MPL/2.0/.

FROM debian:jessie-slim

ENV HG_RELEASE=4.4 \
    VCT_VERSION=tip

# Build/install dependencies
RUN apt-get update
RUN apt-get install -y python ca-certificates curl bash php5-common php5-cli php5-curl php5-json git build-essential python-pip python-dev git vim
RUN pip install simplejson \
    && pip install mercurial==4.5

# add a non-privileged user
RUN addgroup --gid 1000 phab \
    && adduser --uid 1000 --ingroup phab --shell /bin/bash phab

WORKDIR /home/phab

RUN hg clone https://hg.mozilla.org/hgcustom/version-control-tools -r $VCT_VERSION
COPY arcconfig version-control-tools/.arcconfig

RUN git clone https://github.com/glandium/git-cinnabar.git --branch next --single-branch
COPY cinnabar.patch git-cinnabar
RUN cd git-cinnabar && patch -p1 cinnabar.patch && cd ..
ENV PATH="${PATH}:/home/phab/git-cinnabar"
RUN git cinnabar download
RUN git clone hg::https://hg.mozilla.org/hgcustom/version-control-tools vct-cinnabar
COPY arcconfig vct-cinnabar/.arcconfig

COPY hgrc .hgrc
COPY gitconfig .gitconfig

# Install forked Arcanist.
RUN mkdir -p phabricator \
    && cd phabricator \
    && git clone https://github.com/phacility/libphutil.git \
    && git clone https://github.com/zalun/arcanist.git --branch cinnabarc --single-branch

COPY arcrc .arcrc
RUN chmod 600 .arcrc \
    && echo "export PATH=/home/phab/phabricator/arcanist/bin:/home/phab/git-cinnabar:\$PATH" >> .bash_profile \
    && echo "export EDITOR=vi" >> .bash_profile \
    && chown -R phab:phab /home/phab

USER phab
CMD ["/bin/bash", "-l"]
