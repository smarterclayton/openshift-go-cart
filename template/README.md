OpenShift Go Cartridge
======================

Runs [Go](http://golang.org) on [OpenShift](https://openshift.redhat.com/app/login) using downloadable cartridge support. 

Once the app is created, you'll have a ".godir" file in the root of your repo. The single line is to tell the cartridge what the package of your Go code is.  A typical .godir file might contain:

    github.com/smarterclayton/goexample

which would tell OpenShift to place all of the files in the root of the Git repository inside of the <code>github.com/smarterclayton/goexample</code> package prior to compilation.

When you push code to the repo, the cart will compile your package into <code>$OPENSHIFT_REPO_DIR/bin/</code>, with the last segment of the .godir being the name of the executable.  For the above .godir, your executable will be:

    $OPENSHIFT_REPO_DIR/bin/goexample

If you want to serve web requests (vs. running in the background), you'll need to listen on the ip address and port that OpenShift allocates - those are available as HOST and PORT in the environment.

This default "web.go" file is a simple "hello, world" web service. 

Any log output will be generated to <code>$OPENSHIFT_GO_LOG_DIR</code> on your OpenShift gear


Build
-----

When you push code to your repo, a Git postreceive hook runs and invokes a compile script.  This attempts to download the Go compiler environment for you into $OPENSHIFT_GO_DIR/cache.  Once the environment is setup, the cart runs

    go get -tags openshift ./...

on a working copy of your source. 
The main file that you run will have access to two environment variables, $HOST and $PORT, which contain the internal address you must listen on to receive HTTP requests to your application.

