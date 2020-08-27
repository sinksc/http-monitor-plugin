# http-monitor-plugin (Linux/Unix)

Rundeck plugin to monitor HTTP applications for a specific string in the HTTP Response to a GET/POST Request. Simply submits a GET/POST Request using curl and checks the Response to see if there is a match on the supplied test string. If it finds it the step completes successfully. If it doesn't, it fails.
If you are responsible for monitoring an HTTP application for availability, you can create a job using this plugin and schedule it to check your site on a recurring basis.

## Usage

### http / monitor

For a given HTTP URL and string to look for in the response, if that string isn't found in the response it will trigger a failure. This can in turn trigger Notifications or more advanced error processing.

### Pre-Requisites

#### Remote node setup

The executing node's SSH server must be configured to accept RD_* environment variables. This can be read about [here](<https://linux.die.net/man/5/sshd_config>).

Summary, on the executing node:

1. Edit `/etc/ssh/sshd_config` and add this:

    ```shell
    # Accept Rundeck environment variables
    AcceptEnv RD_*
    ```

2. Restart sshd service:

    `sudo service sshd restart`
    
##### Note:

As of Rundeck 3.3, RD_* remote environment variables will no longer be sent by default when using JSCH node executor. You can turn them back on with a configuration change. To enable it for all nodes in all projects, add `framework.ssh-send-env=true` to your `framework.properties` file. [For more options and information, see here](https://docs.rundeck.com/docs/upgrading/upgrading-to-rundeck-3.3.html#jsch-node-executor-timeouts-and-environment-variables).

#### Rundeck setup

If you plan to pass a username:password for server authentication, create an item in Rundeck's [Key Storage](<https://www.rundeck.com/blog/use-rundecks-key-storage-to-manage-passwords-and-secrets>) that houses this entry in the format _username:password_

#### Step inputs

The following fields can passed to the step:

| Input Name | Purpose | Example | Required? |
|:----------:|:-------:|:-------:|:---------:|
| url | URL to submit Request against | _https://some-url.com_ | Y |
| test-string | String to look for in the HTTP Response. No special characters. | _Homepage loaded_ | Y |
| method | Which HTTP method to use in the Request. POST is default. | _GET_ | Y |
| user | Provide a username:password to use for server authentication | _Key Storage entry formatted username:passwword_ | N |
| parameters | Key=value parm list separated by '&' to be passed to curl using the --data flag | _key1=value&key2=value_ | N |

Given the input above, a `curl` command will be issued like this:

```shell
curl --fail --insecure --cookie-jar /dev/null --location --request <method> <url> [--user <user>] [--data <parameters>]
```

The result will be searched to see if it contains the test-string. __Note__ that special chars will be removed from the test-string before searching (spaces are OK).  

#### Failure Codes

This step returns the following exit codes:

| Exit Code |  Reason  |
|:----------:|:-------- |
|      0     | Success. The test string was found in the Response. |
|      1     | Fail. The test string was not found in the Response. |
|      2     | Invalid arguments specified. |
|      3     | General curl command failure (output will display error details). |

## Building

1. To build the plugin, simply zip it but exclude the .git files:

    ```shell
    zip -r http-monitor-plugin.zip http-monitor-plugin -x *.git*
    ```

2. Then copy the zip file to the plugins directory:

    ```shell
    cp http-monitor-plugin.zip $RDECK_BASE/libext
    ```
