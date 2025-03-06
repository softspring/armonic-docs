# Admin Accounts

## Configuration

**Configure routes**

    # config/routes.yaml
    
    _sfs_accounts_:
        resource: "@SfsAccountBundle/Resources/config/routing/admin_accounts.yaml"
        prefix: "/admin/accounts"
        
### List accounts

**Change filter form**

    # config/packages/sfs_account.yaml or config/services.yaml
    
    services:
        Softspring\AccountBundle\Form\Admin\AccountListFilterFormInterface:
            alias: 'App\Form\Admin\AccountListFilterForm'


        
