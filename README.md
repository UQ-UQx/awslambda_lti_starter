# AWS Lambda LTI Starter with Zappa

Instructions for creating an LTI that runs on AWS Lambda, uses Flask and is deployed with Zappa (https://github.com/Miserlou/Zappa)

## Get Zappa up and running
$ mkdir awslambda_lti
$ cd awslambda_lti
$ virtualenv env
$ source env/bin/activate
$ pip install flask zappa pylti

## Create a file called lti_app.py 
$ nano lti_app.py 

```
from flask import Flask
import settings
from pylti.flask import lti

app = Flask(__name__)
app.secret_key = settings.secret_key
app.config.from_object(settings.configClass)

def return_error(msg):
    return render_template('error.htm', msg=msg)


def error(exception=None):
    return return_error('''Authentication error,
        please refresh and try again. If this error persists,
        please contact support.''')

# LTI Launch
@app.route('/launch', methods=['POST', 'GET'])
@lti(error=error, request='initial', role='any', app=app)
def launch(lti=lti):
    """
    Returns the launch page
    request.form will contain all the lti params
    """

    # example of getting lti data from the request
    # let's just store it in our session
    session['lis_person_name_full'] = request.form.get('lis_person_name_full')

    # Write the lti params to the console
    return render_template('launch.htm', lis_person_name_full=session['lis_person_name_full'])


# We only need this for local development.
if __name__ == '__main__':
    app.run()
```

## Create a Flask settings file and customise the CONSUMER_KEY and secret
```
CONSUMER_KEY = 'Enter consumer key'
SHARED_SECRET = 'Enter secret key'
# Configuration for LTI
PYLTI_CONFIG = {
    'consumers': {
        CONSUMER_KEY: {
            "secret": "SHARED_SECRET"
        }
    },
    'roles': {
        # Maps values sent in the lti launch value of "roles" to a group
        # Allows you to check LTI.is_role('admin') for your user
        'admin': ['Administrator', 'urn:lti:instrole:ims/lis/Administrator'],
        'student': ['Student', 'urn:lti:instrole:ims/lis/Student']
    }
}

# Secret key used for Flask sessions, etc. Must stay named 'secret_key'.
# Can be any randomized string, recommend generating one with os.urandom(24)
secret_key = "Enter Flask secret key"
```

## Setup Zappa
$ zappa init

Enter the details Zappa asks eg an S3 Bucket and the app file. A json file is created.

$ nano zappa_settings.json

```
{
    "dev": {
        "app_function": "my_app.app",
        "aws_region": "us-east-1",
        "profile_name": "default",
        "project_name": "awslambdalti",
        "runtime": "python3.6",
        "s3_bucket": "awslambdaltitest"
    }
}
```
## Deploy to AWS Lambda
$ zappa deploy dev

## Update AWS Lambda
$ zappa update dev
