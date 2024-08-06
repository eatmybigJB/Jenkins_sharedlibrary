```python
        {
            "Effect": "Allow",
            "Action": [
                "cognito-idp:AdminCreateUser",
                "cognito-idp:AdminUpdateUserAttributes",
                "cognito-idp:AdminGetUser"
            ],
            "Resource": "arn:aws:cognito-idp:<region>:<account-id>:userpool/<user-pool-id>"
        },
