# dify-mulit-workspace
二次开发dify实现多workspace
修改两处代码

api/services/account_service.py
```python
 @staticmethod
    def create_owner_tenant_if_not_exist(account: Account,
                                         name: Optional[str] = None,
                                         is_setup: Optional[bool] = False):
        """Check if user have a workspace or not"""
        # 增加对owner的判断
        available_ta = (TenantAccountJoin.query.filter_by(
            account_id=account.id).filter_by(role="owner").order_by(
                TenantAccountJoin.id.asc()).first())

        if available_ta:
            return
        """Create owner tenant if not exist"""
        if not FeatureService.get_system_features(
        ).is_allow_create_workspace and not is_setup:
            raise WorkSpaceNotAllowedCreateError()

        if name:
            tenant = TenantService.create_tenant(name=name, is_setup=is_setup)
        else:
            tenant = TenantService.create_tenant(
                name=f"{account.name}'s Workspace", is_setup=is_setup)
        TenantService.create_tenant_member(tenant, account, role="owner")
        account.current_tenant = tenant
        db.session.commit()
        tenant_was_created.send(tenant)
```

api/services/feature_service.py
```python
@classmethod
    def _fulfill_system_params_from_env(cls,
                                        system_features: SystemFeatureModel):
        system_features.enable_email_code_login = dify_config.ENABLE_EMAIL_CODE_LOGIN
        system_features.enable_email_password_login = dify_config.ENABLE_EMAIL_PASSWORD_LOGIN
        system_features.enable_social_oauth_login = dify_config.ENABLE_SOCIAL_OAUTH_LOGIN
        system_features.is_allow_register = dify_config.ALLOW_REGISTER
        system_features.is_allow_create_workspace = True  # 默认为true dify_config.ALLOW_CREATE_WORKSPACE
```
