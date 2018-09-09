
#! /bin/bash
# this an expansion of https://github.com/Integralist/Shell-Scripts/blob/master/aws-cli-assumerole.sh
# Dependencies:
#   brew install jq
#   set aws_access_key_id aws_secret_access_key role_arn mfatoken



# Setup:
#   chmod +x ./assumerole.sh
#
# Execute:
#    ./assumerole.sh
#    supply name of the session (terraforming  for example)
#    MFA token value
# Result: credentials file is craeted
# use profile  & terraforming ec --prodfile terraforming




# create new config file


rm credentials

echo "[default]" > credentials
#file handle to close file ready for reading in
3<> credentials

echo "aws_access_key_id = $aws_access_key_id " >> credentials
echo "aws_secret_access_key = $aws_secret_access_key" >> credentials
echo "region = $region" >> credentials


read -p "name the role : " name
read -p "Enter token : " mfatoken

# close file
exec 3>&-

temp_role=$(aws sts assume-role \
                    --role-arn "$role_arn" \
                    --role-session-name "anything" \
                    --serial-number  "$serial_number" \
                   --token-code $mfatoken \
                    --duration-seconds 900)



echo [$name] >> credentials
echo aws_access_key_id = $(echo $temp_role | jq .Credentials.AccessKeyId | xargs) >> credentials
echo aws_secret_access_key = $(echo $temp_role | jq .Credentials.SecretAccessKey | xargs) >> credentials
echo aws_session_token = $(echo $temp_role | jq .Credentials.SessionToken | xargs) >> credentials
