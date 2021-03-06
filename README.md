# Azure AD Login to AWS

The `ad-aws-login.sh` script fetches temporary AWS credentials with Azude AD
login (https://myapps.microsoft.com).

So far this has been used **only on OS X**.

The script launches Chrome with a separate session and helps you through the
login with a dedicated Chrome extension. Because this is a new session,
Chrome will ask you about default browser etc. And if you choose "Remember
me" on the first login, you don't need to enter your username all the time.

The motivation for all this was to get rid of the host of dependencies
[aws-azure-login](https://github.com/sportradar/aws-azure-login) requires.
Now the codebase is compact enough that you can read it through, and verify
that is not malicious. (apart from minified aws sdk which you can download
yourself from https://github.com/aws/aws-sdk-js/releases)

## How it works

The script adds temporary credentials to `~/.aws/credentials`. You can then
use those credentials by setting your `AWS_PROFILE` environment variable
accordingly.

Logging in gets you into the role you've been assigned to by default. To
assume other roles, it's recommended you have a section in `~/.aws/config`
for each role you're going to use. Something like this:

```
[profile <profile name for role to assume>]
region=<your region>
source_profile=<the profile name in ~/.aws/credentials you log in to>
role_arn=<arn for the role to assume>
app=A substring of the app name shown in myapps.microsoft.com to launch. Case-insensitive. Must be url encoded.
```

## Usage

Option `--profile` is mandatory.

```
Usage: ${0} [OPTIONS]

  Simple script that fetches temporary AWS credentials with Azude AD login
  (https://myapps.microsoft.com).

Options:
  --profile  TEXT    The name of the profile in ~/.aws/credentials to update.
  --app      TEXT    A substring of the app name shown in myapps.microsoft.com
                     to launch. Case-insensitive. Must be url encoded.
  --duration INTEGER How many hours the temporary credentials are valid.
  --role-arn TEXT    AWS IAM Role to assume with AD credentials.
```

## Example

Log in to your sandbox account. Assuming the link to the sandbox account in
myapps.microsoft.com is called "AWS test", then `--app` argument should be
"AWS%20test". If `--app` or `--role-arn` is missing from parameters, you are asked
to select them in browser. Can be written in the `.aws/config` file aswell. `Write temporary credentials to `~/.aws/credentials`
under a profile called `sandbox`:

```
./ad-aws-login.sh --profile sandbox --app "AWS%20test" --duration-hours 4 --role-arn arn:aws:iam::123456789012:role/Developer
# Unset any lingering AWS credentials from environment
unset AWS_SESSION_TOKEN AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY
# Set active profile to andbox
export AWS_PROFILE=sandbox
```

You can also configure your profile in `.aws/config`. Do note that profiles should be separated with a newline
```
[profile test-admin]
region=eu-west-1
app=AWS%20test
role_arn=arn:aws:iam::123456789012:role/Developer

[profile another-profile]
...
```

**Note** Your account probably has some maximum session duration. Trying to
use longer `--duration-hours` will cause the script to get stuck.

**Pro tip:** Put this in a bash alias or script.

## Useful commands

If you have [fzf](https://github.com/junegunn/fzf) installed, it
will be beneficial.

## TODO

* Downloading the credentials from chrome as a file is not that neat. Is there
  some other communication channel? What changes to chrome extension would
  enable it to write the file directly? (`~/.aws` is about as safe as
  `~/Downloads` so I think this is a matter of style)
