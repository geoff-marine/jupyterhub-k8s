# jupyterhub-k8s

This is a deployment to micro k8s of Jupyterhub with the following features:
1. Jupyterhub native authenicator configured
2. A shared folder available to all users

## Micro k8s config
Need to have enabled storage, ingress, dns, ha-cluster<br>
I also enabled metalb to use as a loadbalancer.  The Jupyterhub helm chart expects a k8s loadbalancer rather than a nodeport etc.

## Jupyterhub install

### Shared folder

Have not added this to chart yet so needs to be added manually with kubectl first before running the helm chart

    kubectl create namespace jupyterhub
    kubectl -n jupyterhub apply -f pvc-shared-data.yaml

### Jupyterhub - helm

I had helm3 installed on my machine so did not use the helm option that comes with microk8s<br>
As of right now the current stable version has a bug so instead I targetted a known good dev build: 0.11.1-n270.he08bb7f8<br>

Inital install - this creates a release called jupyterhubv1 in a namespace jupyterhub


    helm upgrade --cleanup-on-fail   --install jupyterhubv1 jupyterhub/jupyterhub   --namespace jupyterhub   --create-namespace   --version=0.11.1-n270.he08bb7f8   --values config.yaml <br>

For upgrade/config changes:


    helm upgrade --cleanup-on-fail jupyterhubv1 jupyterhub/jupyterhub   --namespace jupyterhub  --version=0.11.1-n270.he08bb7f8   --values config.yaml

## Operating - User Managment

### Intial Admin User

In config.yaml add the name of your admin user under the admin_users block<br>
The first time this admin user logs on they will need to sign up using the signup option in the log in screen.  The username must be the same as in the config.yaml - the user can then set their password. Once this is done return to the signin page and login using the username and password that has just been set.

### Adding more users
Users can be added two ways
1. User creates account and waits for Admin to approve
2. User is added by Admin user

#### User creates account and waits for Admin to approve
User uses the signup link on the login page to create an account.  It then needs to be authorised by an Admin before this user can access Jupyterhub.

#### User added by Admin
Admin user adds the new user through the admin tab in the control panel.  Admin adds the users username.  User still needs to use the signup link in the login page to create a password.  Once the user has signed up, an Admin still needs to autorise the user.
<p>No real advantage in an Admin creating an account for an user.

#### Authorising a new user
Can only be done by an admin user.  No direct link in the GUI at the moment to the Auth page so after logging in, the Admin user need to go to `<jupyterhub url>`/hub/authorize to allow or reject an new user.

## Operating - Shared folder
Every user has access to this folder.  If you want to prevent other users writing the contents of this folder you use chmod in a terminal session in jupyterhub but note that any other user can also use chmod on this folder.

## Operating - Git
Git is preinstalled in the image that we are using - users can access and configure through a terminal in jupyterhub.

