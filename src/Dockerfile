# start from python base image
FROM python:3.10

# change working directory
WORKDIR /code

# add requirements file to image
COPY ./requirements.txt /code/requirements.txt

# install python libraries
RUN pip install --no-cache-dir torch==2.11.0 --extra-index-url https://download.pytorch.org/whl/cpu

RUN pip install --no-cache-dir --upgrade --timeout=300 -r /code/requirements.txt

# add python code
COPY . /code

# specify default commands
CMD ["fastapi", "run", "main.py", "--port", "80"]