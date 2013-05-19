OpenShift Go Cartridge
======================

Runs [Go](http://golang.org) on [OpenShift](https://openshift.redhat.com/app/login) using downloadable cartridge support.  To install to OpenShift from the CLI (you'll need version 1.9 or later of rhc), run:

    rhc create-app mygo http://cartreflect-claytondev.rhcloud.com/reflect?github=smarterclayton/openshift-go-cart

Once the app is created, you'll need to create and add a ".godir" file in your repo to tell the cartridge what the package of your Go code is.  A typical .godir file might contain:

    github.com/smarterclayton/example

which would tell OpenShift to place all of the files in the root of the Git repository inside of the 'github.com/smarterclayton/example' package prior to compilation.

When you push code to the repo, the cart will compile your package into $OPENSHIFT_REPO_DIR/bin/, with the last segment of the .godir being the name of the executable.  For the above .godir, your executable will be:

    $OPENSHIFT_REPO_DIR/bin/example

If you want to serve web requests (vs. running in the background), you'll need to listen on the ip address and port that OpenShift allocates - those are available as HOST and PORT in the environment.

Here's a sample go file which would print "hello, world" when someone hits your web application:

    package main

    import (
      "fmt"
      "net/http"
      "os"
    )

    func main() {
      http.HandleFunc("/", hello)
      fmt.Println("listening...")
      err := http.ListenAndServe(os.Getenv("HOST")+":"+os.Getenv("PORT"), nil)
      if err != nil {
        panic(err)
      }
    }

    func hello(res http.ResponseWriter, req *http.Request) {
      fmt.Fprintln(res, "hello, world")
    }

Any log output will be generated to $OPENSHIFT_GO_DIR/logs/go.log


How it Works
------------

When you push code to your repo, a Git postreceive hook runs and invokes the bin/compile script.  This attempts to download a Go environment for you into $OPENSHIFT_GO_DIR/cache.  Once the environment is setup, the cart runs

    go get -tags openshift ./...

on a working copy of your source.  The main file that you run will have access to two environment variables, $HOST and $PORT, which contain the internal address you must listen on to receive HTTP requests to your application.


Credits
-------

The bin/compile script is based on the [Heroku Go buildpack](https://github.com/kr/heroku-buildpack-go), adapted for OpenShift cartridges.