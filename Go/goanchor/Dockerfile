FROM golang:1.22.4-bookworm

#####################
# COMMON
#####################
ARG NVIM_VERSION=v0.10.0

# Use bash instead of sh
# RUN rm /bin/sh && ln -s /bin/bash /bin/sh

# Dependencies
RUN apt-get update && apt-get install -y \
    unzip \
    gzip \
    curl \
    wget \
    git \
    gnupg2 \
    ripgrep \
    fd-find \
    locales \
    bash-completion \
    tmux \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Tmux
ENV TERM=screen-256color
RUN git clone https://github.com/stiffy-wood/stiffy-tmux.git && \
    ln -s stiffy-tmux/tmux.conf ~/.tmux.conf

# Git
RUN git config --global user.name "Stiffy" && \
    git config --global user.mail "johnnykgen@gmail.com" 
ENV GITHUB_TOKEN=${GITHUB_TOKEN}

# Neovim
RUN wget https://github.com/neovim/neovim/releases/download/${NVIM_VERSION}/nvim-linux64.tar.gz && \
    mkdir -p /opt/nvim && \
    tar xzvf nvim-linux64.tar.gz -C /opt/nvim --strip-components 1 && \
    rm nvim-linux64.tar.gz

RUN echo 'export PATH="/opt/nvim/bin:$PATH"' >> ~/.bashrc
ENV PATH="${PATH}:/opt/nvim/bin"

# Neovim config
RUN git clone https://github.com/stiffy-wood/stiffy-nvim.git /root/.config/nvim
RUN nvim --headless +PlugInstall +qall

# Workspace
VOLUME /repos
ENV LANG=en_US.UTF-8

#####################
# COMMON END
#####################

# Dependencies
RUN wget -qO - 'https://proget.makedeb.org/debian-feeds/prebuilt-mpr.pub' | gpg --dearmor | tee /usr/share/keyrings/prebuilt-mpr-archive-keyring.gpg 1> /dev/null && \
    echo "deb [arch=all,$(dpkg --print-architecture) signed-by=/usr/share/keyrings/prebuilt-mpr-archive-keyring.gpg] https://proget.makedeb.org prebuilt-mpr bookworm" | tee /etc/apt/sources.list.d/prebuilt-mpr.list && \
    apt update && \
    apt-get install -y \
    just && \
    rm -rf /var/lib/apt/lists/*

RUN just --completions bash >> ~/.bashrc
RUN nvim --headless -c "MasonInstall gopls gotests goimports golines" +qall

# Workspace setup
WORKDIR /repos/goanchor

RUN echo "export PATH=/usr/local/go/bin:$PATH" >> ~/.bashrc

RUN echo \
    "#!/bin/sh\n" \
    "if [ ! -d \"/repos/goanchor/.git\" ]; then\n" \
    "    git clone https://github.com/stiffy-wood/goanchor.git /repos/goanchor;\n "\
    "fi\n" \
    "tmux -u" > /usr/local/bin/entrypoint.sh && \
    chmod +x /usr/local/bin/entrypoint.sh
