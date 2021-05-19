 # Demo Bug
 
**Problem:** Assume role with mfa not working.

**Reason:** I didn't log out before running "aws configures sso", which merged sso details into the mfa user in the "~/.aws/config" file.

It looked like it was still allowing command, but that's because it was defaulting to the SSO user.  

When I ran aws sso logout the command stopped working and an SSO Token error appeared.

**Solution:** Manually edit "~/.aws/config"

## Retry that section of the demo:

### Part 1:

`% aws s3 ls`

Unable to locate credentials. You can configure credentials by running "aws configure".

    % export AWS_PROFILE=assume-role-user-mfa
    % aws sts get-caller-identity
    {
        "UserId": "AIDAYU4J3MOEXAAAABBBB",
        "Account": "594601234567",
        "Arn": "arn:aws:iam::594601234567:user/assume-role-user-mfa"
    }
    % aws s3 ls
    An error occurred (AccessDenied) when calling the ListBuckets operation: Access Denied

This is expected because I am not logged in with MFA.

When I use the sts-w-mfa.sh script with my one-time token, it now allows an s3 ls.

    % source sts-w-mfa.sh 396514
    Usage: source sts-w-mfa.sh <token-code>
    396514
    {
        "UserId": "AIDAYU4J3MOEXAAAABBBB",
        "Account": "594601234567",
        "Arn": "arn:aws:iam::594601234567:user/assume-role-user-mfa"
    }
    ASIAYU4J3MOEAAAABBBB
    
    % aws s3 ls
    2021-05-16 09:24:56 test-bucket-for-meetup-demo

### Part 2:

Assume role with ext_id, role requires mfa

    % source sts-logout.sh
    Usage: source sts-logout.sh
    Logout Script Complete
    % export AWS_PROFILE=assume-role-user-mfa
    % aws sts get-caller-identity
    {
        "UserId": "AIDAYU4J3MOEXAAAABBBB",
        "Account": "594601234567",
        "Arn": "arn:aws:iam::594601234567:user/assume-role-user-mfa"
    }
    
    % source sts-assume-role.sh arn:aws:iam::714741234567:role/assume_role_with_ext_id_with_mfa chi_meetup_ext_id
    Usage: source sts-assume-role.sh <role> (<external-id>)

    An error occurred (AccessDenied) when calling the AssumeRole operation: User: arn:aws:iam::594601234567:user/assume-role-user-mfa is not authorized to perform: sts:AssumeRole on resource: arn:aws:iam::714741234567:role/assume_role_with_ext_id_with_mfa

Again the deny is expected, because I didn't authenticate with mfa.

Now when I use cli MFA, I can assume the remote role and list the s3 buckets in that account.

    % source sts-w-mfa.sh 525087
    Usage: source sts-w-mfa.sh <token-code>
    525087
    {
        "UserId": "AIDAYU4J3MOEXAAAABBBB",
        "Account": "594601234567",
        "Arn": "arn:aws:iam::594601234567:user/assume-role-user-mfa"
    }
    ASIAYU4J3MOEAAAABBBB
    
    % source sts-assume-role.sh arn:aws:iam::714741234567:role/assume_role_with_ext_id_with_mfa chi_meetup_ext_id
    Usage: source sts-assume-role.sh <role> (<external-id>)
    {
        "UserId": "AROA2M2TAOCWRAAAABBBB:assumed-role-session",
        "Account": "714741234567",
        "Arn": "arn:aws:sts::714741234567:assumed-role/assume_role_with_ext_id_with_mfa/assumed-role-session"
    }
    
    % aws s3 ls
    2021-05-08 09:59:38 my-test-bucket-0508
    2021-05-16 10:14:20 my-test-bucket-5-16

Join me next month, where I will demo this live.
