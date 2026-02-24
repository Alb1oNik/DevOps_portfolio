---------------
How to create new user
---------------

1. Create user "username" with home directory and set /bin/bash

        sudo useradd -m -s /bin/bash username

2. Add user to sudo

        sudo usermode -aG sudo username

3. Checkin user groups

        id username

4. Create start password (unsafe like teks - using only for inicialization)

        echo 'user:Somepasswd' | sudo chpasswd

5. Forced password change at first login for username

        sudo passwd --expire username