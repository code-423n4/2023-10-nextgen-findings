The contract does not emit events when admins are added or removed. This makes it harder to track changes in admin permissions.

 function registerAdmin(address _admin, bool _status) public onlyOwner {
        adminPermissions[_admin] = _status;
    }
