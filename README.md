# aws-cloudformation-dev-rest

## Synopsis

Using the AWS Cloudformation template in this repository,
a developer can bring up an AWS stack to interact with Senzing's API server
in a secure manner.

## Overview

The following diagram illustrates a Senzing stack that would be brought up by the AWS CloudFormation template.

Using this stack, a developer could interact with Senzing's api server programmatically.

![overview diagram](assets/overview_diagram.png)

## Contents

1. [Pre-requisites](#pre-requisites)
1. [Deploy](#deploy)
1. [Interact with Senzing API Server using SSL client authentication](#interact-with-senzing-api-server-using-ssl-client-authentication)
1. [Example application](#example-application)
1. [References](#references)

## Pre-requisites

1. Deploy [aws-cloudformation-database-cluster cloudformation stack](https://github.com/Senzing/aws-cloudformation-database-cluster)
1. Install [adoptopenjdk 11](https://adoptopenjdk.net/archive.html) (it must be this specific java version)
1. Install [git](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/install-git.md)

## Deploy

1. :warning: **Warning:** This Cloudformation deployment will accrue AWS costs.
   With appropriate permissions, the
   [AWS Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/)
   can help evaluate costs.
1. Visit [AWS Cloudformation with dev-rest template](https://console.aws.amazon.com/cloudformation/home#/stacks/new?stackName=sz-dev&templateURL=https://s3.amazonaws.com/public-read-access/aws-cloudformation-dev-rest/cloudformation.yaml)
1. At lower-right, click on "Next" button.
1. In **Specify stack details**
    1. In **Parameters**
        1. In **Senzing installation**
            1. Accept the End User License Agreement
            1. Adjust the Senzing version, if necessary
            1. If using more than 100k records, input base64 encoded license string
        1. In **Identify existing database stack**
            1. Enter the stack name of the previously deployed
               [aws-cloudformation-database-cluster](https://github.com/Senzing/aws-cloudformation-database-cluster)
               Cloudformation stack
               Example:  `senzing-db`
        1. In **Security**

           :warning: The following assumes a "Server store file" and "Client trust store file"
           have been created.
           If not, see
           [How to generate keystores for SSL client authentication](#how-to-generate-keystores-for-ssl-client-authentication).

            1. Enter your email address.
                1. Example: `me@example.com`
            1. Enter the permitted IP address block
        1. In **Security responsibility**
            1. Understand the nature of the security in the deployment.
            1. Once understood, enter "I AGREE".
    1. At lower-right, click "Next" button.
1. In **Configure stack options**
    1. At lower-right, click "Next" button.
1. In **Review senzing-basic**
    1. Near the bottom, in **Capabilities**
        1. Check ":ballot_box_with_check: I acknowledge that AWS CloudFormation might create IAM resources."
    1. At lower-right, click "Create stack" button.

## Interact with Senzing API Server using SSL client authentication

1. :pencil2: Retrieve the Senzing API Server URL from the cloudformation stack that was brought up.
   It can be found in the output tab, under the key "UrlApiServer".
   This is what it looks like in the AWS Cloudformation management console:

   ![api url](assets/cloudformation_output_api.png)

<br/>Example:

    ```console
    export SENZING_API_SERVER_URL=https://XXXXXXXX.amazonaws.com/api
    ```

1. To interact directly with the Senzing API server,
   use the `--cert` and `--cert-type` options of
   `curl` to authenticate to the API server.
   <br/> 
   <br/>Example:

    ```console
    curl \
        --insecure \
        ${SENZING_API_SERVER_URL}/heartbeat \
        --cert ${KEYTOOL_CLIENT_STORE_FILE}:${KEYTOOL_CLIENT_PASSWORD} \
        --cert-type P12
    ```

## Example applications

Two example applications were created to illustrate how an application can authenticate with the Senzing's API Server. The cli application can be found [here](examples/cli_demo.py) and the web application can be found [here](examples/webapp_demo.py)

To run the example python applications:

1. Clone repository.
   First, set these environment variable values:

    ```console
    export GIT_ACCOUNT=senzing
    export GIT_REPOSITORY=aws-cloudformation-dev-rest
    export GIT_ACCOUNT_DIR=~/${GIT_ACCOUNT}.git
    export GIT_REPOSITORY_DIR="${GIT_ACCOUNT_DIR}/${GIT_REPOSITORY}"
    ```

   then using the environment variables values just set,
   follow steps in [clone-repository](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/clone-repository.md)
   to install the Git repository.

1. Export the following variables.
   <br/> 
<br/>Example:

    ```console
    export CLIENT_STORE_PATH=${KEYTOOL_CLIENT_STORE_FILE}
    export CLIENT_STORE_PASSWORD=${KEYTOOL_CLIENT_PASSWORD}
    export API_URL=${SENZING_API_SERVER_URL}
    ```

1. Use the following commands to run the example web application. 
<br/>Note: Homebrew users may see deprecation warnings like "DEPRECATION: Configuring installation scheme with distutils ...". Do not worry and continue on. To understand the warnings, refer to [this](https://github.com/Homebrew/homebrew-core/issues/76621)
<br/> 
<br/>Example:

    ```console
    cd ${GIT_REPOSITORY_DIR}/examples
    pip install -r requirements.txt
    export FLASK_APP=webapp_demo
    flask run
    ```

1. To get the example web application to interact the Senzing's api server,
   simply send the following `curl` commands.
   <br/> 
<br/>Example:

    ```console
    curl http://127.0.0.1:5000

    curl http://127.0.0.1:5000/test-query
    ```

   Or in a browser visit:

   - [http://127.0.0.1:5000](http://127.0.0.1:5000)
   - [http://127.0.0.1:5000/test-query](http://127.0.0.1:5000/test-query)

1. Use the following command to run the example cli application.

    ```
    python examples/cli_demo.py
    ```

1. For more APIs, please refer to this [swagger documentation](https://editor.swagger.io/?url=https://raw.githubusercontent.com/Senzing/senzing-rest-api/master/senzing-rest-api.yaml).


## References

To understand mutual TLS authentication better, refer to the resources here:

- [What is Mutual TLS Authentication?](https://www.cloudflare.com/learning/access-management/what-is-mutual-tls/)
- [Difference between key store and trust store](https://www.baeldung.com/java-keystore-truststore-difference)
