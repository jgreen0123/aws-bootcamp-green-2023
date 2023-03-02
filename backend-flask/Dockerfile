FROM python:3.10-slim-buster

# Inside Container
# Make a new folder inside the container
WORKDIR /backend-flask

# Outside Container -> Inside Container
# Contains the libraries we want to install to run the app
COPY requirements.txt requirements.txt

# Inside Container
# Installs the python libraries used for the app
RUN pip3 install -r requirements.txt

# Outside Container -> Inside Container
# Copies everything in the backend-flask directory to inside the container
COPY . .

# Set Enviroment variables inside the container
ENV FLASK_ENV=development

EXPOSE ${PORT}

# Command to run flask
# python3 -m flask run --host=0.0.0.0 --port=4567
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]