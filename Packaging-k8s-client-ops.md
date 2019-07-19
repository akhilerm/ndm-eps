### Packaging k8s Client and Disk/BD-store

Currently NDM uses methods of `Controller` struct to perform k8s api operations like list, create, update and delete. In reality only the client inside controller is used by these methods.(defined in `cmd/ndm_daemonset/controller/*store.go` files). This results in same code being rewritten for the NDM operator CRUD operations. 

This can be avoided by creating a client struct which wraps the k8s client and defining methods for this struct for all the crud operations. This will greatly reduce the amount of code to be maintained and make the repo more streamlined and readable. All these client methods, generating client etc will have to be moved to a new package. 