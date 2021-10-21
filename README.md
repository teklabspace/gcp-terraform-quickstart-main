# gcp-terraform-quickstart
Demo solution of load-balanced nginx servers, exploring GCP and using Terraform for infra-as-code

# Terraform "Hello, World" example

This folder contains a "Hello, World" example of a [Terraform](https://www.terraform.io/) file on Google Cloud Platform (GCP).

This Terraform file deploys a single server on Google Cloud Platform (GCP) using the shortest script.

## Requirements

* You must have [Terraform](https://www.terraform.io/) installed on your computer.
* You must have a [Google Cloud Platform (GCP)](https://cloud.google.com/) account.
* You must have downloaded a Google Cloud Platform credentials file.
* You must have enabled the Google Compute Engine API.
* It uses the Terraform Google Cloud Provider that interacts with the many resources supported by Google Cloud Platform (GCP) through its APIs.
* This code was written for Terraform 1.x.x.

## Using the code

* Configure your Google Cloud access keys.

  Two ways in order to configure credentials:

  * Configure `GOOGLE_APPLICATION_CREDENTIALS` environment variable. The variable must contain the path to the credentials file.

    To set these variable on Linux, macOS, or Unix, use `export`:

    ```bash
    export GOOGLE_APPLICATION_CREDENTIALS="~/.gcloud/terraform-examples-code.json"
    ```

    To set these variable on Windows, use `set`:

    ```bash
    set GOOGLE_APPLICATION_CREDENTIALS="C:\Users\USERNAME\.gcloud\terraform-examples-code.json"
    ```

  * Configure `GOOGLE_CREDENTIALS` environment variable. The variable must contain the content of the credentials file and not the path to it.

    To set these variable on Linux, macOS, or Unix, use `export`:

    ```bash
    export GOOGLE_CREDENTIALS="$(cat ~/.gcloud/terraform-examples-code.json)"
    ```

* Initialize working directory.

  The first command that should be run after writing a new Terraform configuration is the `terraform init` command in order to initialize a working directory containing Terraform configuration files. It is safe to run this command multiple times.

  ```bash
  terraform init
  ```

* Validate the changes.

  Run command:

  ```bash
  terraform plan
  ```

* Deploy the changes.

  Run command:

  ```bash
  terraform apply
  ```

* Test the deploy.

  When the `terraform apply` command completes, use the Google Cloud console, you should see the new Google Compute instance.

* Clean up the resources created.

  When you have finished, run command:

  ```bash
  terraform destroy
  ```


  Set up Google Cloud Platform (GCP) authentication for Terraform Cloud


First of all, you will need to set up a service account in your GCP project in order for Terraform Cloud to be able to manage resources for you. Just do the following:

    Log in to the GCP console and switch to the desired project.
    Go to the IAM & Admin → Service accounts section.
    Press the "Create service account" button.
    Specify a meaningful name for your service account and click "Create and continue".
    Specify a role for your service account. For test purposes you can use the Owner role with the maximum permissions. However, in production I would highly recommend to create a separate role for your service account with minimal possible permissions.
    Then click "Done" to finally create the service account.
    Now, select the newly created account from the list and go to the "KEYS" tab.
    Press the "ADD KEY" button and select the "Create a new key" option.
    Select the JSON format and press "CREATE".
    Download the key file to your machine and open it in your favorite text editor.
    The provided key is in multiline JSON format, however, in order to be able to use it in Terraform configuration it should be minified. You can use any JSON minifier that you can trust. Otherwise, you can use a "find & replace" functionality of your text editor to remove all multiline characters. In the end you should receive a JSON document as a single line of text, copy it.

Now, you will need to specify the JSON key in your Terraform configuration. The most straightforward way to do so would be to put it directly in your google provider configuration under the credentials property. However, it is a VERY BAD practice to store such sensitive data in your code. We would do something else instead:

    Add the following variable declaration to your Terraform configuration file:

```bash
variable "gcp_credentials" {
  type = string
  sensitive = true
  description = "Google Cloud service account credentials"
}
```

This will tell Terraform that this input variable actually exists and could be used to configure the stack.

    Then, go to your Terraform Cloud console and switch to the desired workspace. Go to the "Variables" tab.

    Now, press the "Add variable" button and specify the following data:
        * Key: gcp_credentials
        * Value: INSERT YOUR SINGLE-LINE JSON HERE
        * Description: Google Cloud service account credentials
        * Check the "Sensitive" checkbox.

    Click the "Save variable button".

Finally, update your provider configuration to look like this:

```bash
provider "google" {
  project = "my-project-id"
  credentials = var.gcp_credentials
  region = "europe-west3"
  zone = "europe-west3-a"
}
```

Using this approach, your secret JSON key would be securely stored by the Terraform Cloud without the ability for anybody to read it directly (thanks to the "sensitive" option) and the key would be provided to the Google Provider at runtime by the means of Terraform input variable.

However, be advised that it's not a bullet-proof way of storing secrets and there are situation when the content of the variable could be read by a party with the ability to update the configuration and read the log files.

I would also recommend to move other information such as region/zone and project ID from the config file. This will make your stack more reusable and your configuration cleaner (by removing duplication).
