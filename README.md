# AWS CLI credentials in 1Pass
🔑 Simply store sensitive AWS access keys used by CLI in 1Password

You want to use AWS CLI easily but easiest way is to store raw access keys in `.aws/config` file and its anti-pattern i think.
For some time i try to found solution that will be easy to use and secure. Also it would be great if you dont need any random packages to make it work.

And here it is. AWS native thing that work for exampel with Terraform (official 1Password CLI plugin for AWS is not supported) and
you can extend basic profile with another assume roles etc.

Its simple!<br/>
For this we are using only two comnponents
- [AWS CLI](https://aws.amazon.com/cli/) (with [`credential_process`](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sourcing-external.html) option)
- [1Password CLI](https://developer.1password.com/docs/cli/)

Create bash file for example `.aws/credentials-provider.sh` and put inside this very simple code.


```bash
#!/bin/bash

# Take secret vriables from 1Password via CLI (password or touch id will be required)
access_key=$(op item get "my-main-account-access-key" --field "access key id")
secret_access_key=$(op item get "my-main-account-access-key" --field "secret access key")

# JSON output needed by AWS CLI
json_output="{\"Version\": 1, \"AccessKeyId\": \"$access_key\", \"SecretAccessKey\": \"$secret_access_key\"}"

# Print the JSON to stdout
echo "$json_output"
```

And then update your `.aws/config` file like that

```yml
[default]
credential_process = "/Users/my-user/.aws/credentials-provider.sh"

# You can add any special params (for example MFA device)
region = eu-west-1
output = json
```

And thats it! 1Password CLI will get your item named `my-main-account-access-key` and fileds `access key id` and `secret access key`.
Super thing is, that there is no cache that means for every request item will be fetched from 1Pass CLI again that means more security for us.
1Pass CLI will automatically check for additional verification and optionally require password/touch id.

Iam using this also with additional profiles that extends default profile via `source_profile = default` and then iam adding `role_arn` to connect into different accounts for my company/clients etc.
Its good practice to secure bash file and allow only restricted execution!
