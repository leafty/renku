FROM python:3.6

# RUN apk add --update openssl
RUN wget https://github.com/mozilla/geckodriver/releases/download/v0.11.1/geckodriver-v0.11.1-linux64.tar.gz && \
    tar -xvzf geckodriver* && \
    chmod +x geckodriver && \
    mv geckodriver /usr/local/bin && \
    apt-get update -y && \
    apt-get install -y firefox-esr xvfb enchant shellcheck

COPY . /tests

WORKDIR /tests
RUN pip install -r requirements.txt
CMD xvfb-run pytest -v
