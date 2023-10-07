## How to test

Try \
`docker compose up`

Then try \
`docker compose --env-file ./cli_envfile.env up`

## Testing

### Environment
Tests ran on Windows 10 \
`Docker Desktop 4.24.1 (123237)` \
`Docker Compose version v2.22.0-desktop.2`

### Results
In the official docker documentation it uses an example of the --env-file argument usage with the root .env file.

https://docs.docker.com/compose/environment-variables/set-environment-variables/#substitute-with---env-file

From the tests it acts like --env-file default is ".env" when not set.  Thus setting this argument actually prevents the base .env file from loading.  So is the --env-file default ".env"?

If it was then it would also mean that the env vars precedence page is incorrect.

https://docs.docker.com/compose/environment-variables/envvars-precedence/

It suggests that the --env-file cli argument takes precedence over the env_file attribute in the Compose file.  From the tests below it confirms that the --env-file cli argument is the same precedence as the .env file in base folder.

The documentation also does not explain that load order for Parameter expansion does not follow precedence.

Suggested load order from tests:
1. env-file from cli argument or .env file
2. environment attributes
3. env-file attribute (multiple files are loaded in order as written)

From the tests you can see that you can override the environment attributes by using parameter expansion. \
See `TEST_OVERRIDE_ATTRIBUTE`

### Conclusion

Either the documentation needs to be updated or docker compose needs to be updated to reflect what is documented.

#### 1. Fix page about --env-file

https://docs.docker.com/compose/environment-variables/set-environment-variables/#substitute-with---env-file

This page should be changed to reflect that --env-file prevents .env from loading. There is a note that says
> If the --env-file is not used in the command line, the .env file is loaded by default

But it doesn’t say anything about using the --env-file prevents loading the .env file. \
If the default value for --env-file is ".env" then it should be in the documentation.

#### 2. Fix env vars precedence page

https://docs.docker.com/compose/environment-variables/envvars-precedence/

This page says that the --env-file argument in the cli takes precedence over the env_file attribute in the compose file.  The tests show this to be incorrect.  The env_file attribute takes precedence over both the --env-file argument from the cli and the .env file in the base directory.

If its impossible to load the ".env" file in base directory while using the --env-file argument from the CLI then they should be the same number in the precedence list.

#### 3. Load order and Parameter Expansion

https://docs.docker.com/compose/environment-variables/envvars-precedence/ \
https://docs.docker.com/compose/environment-variables/env-file/#parameter-expansion

I suggest information is added to both of these pages around Parameter expansion not applying the same Precedence.  From the tests it seems like its Load order.

Suggested load order from tests:
1. env-file from cli argument or .env file
2. environment attributes
3. env-file attribute (multiple files are loaded in order as written)


### Test 1 Results
```
docker compose up
---
TEST_OVERRIDE_ATTRIBUTE=using_baseenvfile
TEST_ENVFILE2_CHECK_CLIENVFILE=not_found
TEST_ENVFILE1_CHECK_ENVFILE2=not_found
TEST_ENVFILE2_CHECK_BASEENVFILE=base_env_file
TEST_VAR_ENVFILE2=env_file2
TEST_VAR_ENVFILE1=env_file1
TEST_ENVFILE1_CHECK_CLIENVFILE=not_found
TEST_ENVATTRIBUTE_CHECK_ENVFILE1=not_found
TEST_ENVATTRIBUTE_CHECK_ENVFILE2=not_found
TEST_ENVATTRIBUTE_CHECK_BASEENVFILE=base_env_file
TEST_ENVFILE1_CHECK_ENVATTRIBUTE=not_found
TEST_ENVFILE2_CHECK_ENVATTRIBUTE=not_found
TEST_ENVFILE2_CHECK_ENVFILE1=env_file1
TEST_ENVATTRIBUTE_CHECK_CLIENVFILE=not_found
TEST_OVERRIDE_ENV=env_file2_override
TEST_ENVFILE1_CHECK_BASEENVFILE=base_env_file
TEST_VAR_ENVATTRIBUTE=env_attribute
```

### Test 2 Results
```
docker compose --env-file ./cli_envfile.env up
---
TEST_OVERRIDE_ATTRIBUTE=using_clienvfile
TEST_ENVFILE2_CHECK_CLIENVFILE=cli_env_file
TEST_ENVFILE1_CHECK_ENVFILE2=not_found
TEST_ENVFILE2_CHECK_BASEENVFILE=not_found
TEST_VAR_ENVFILE2=env_file2
TEST_VAR_ENVFILE1=env_file1
TEST_ENVFILE1_CHECK_CLIENVFILE=cli_env_file
TEST_ENVATTRIBUTE_CHECK_ENVFILE1=not_found
TEST_ENVATTRIBUTE_CHECK_ENVFILE2=not_found
TEST_ENVATTRIBUTE_CHECK_BASEENVFILE=not_found
TEST_ENVFILE1_CHECK_ENVATTRIBUTE=not_found
TEST_ENVFILE2_CHECK_ENVATTRIBUTE=not_found
TEST_ENVFILE2_CHECK_ENVFILE1=env_file1
TEST_ENVATTRIBUTE_CHECK_CLIENVFILE=cli_env_file
TEST_OVERRIDE_ENV=env_file2_override
TEST_ENVFILE1_CHECK_BASEENVFILE=not_found
TEST_VAR_ENVATTRIBUTE=env_attribute
```
