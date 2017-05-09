# Tips for running faster builds with nodejs and docker

# Avoid npm bloat

Ok so you've got a killer idea and you're going a mind blowing web application, so you do a bit of reading and decide to use node and docker. You're now finally ready to get started. You know your app is going to need: a web framework, it's going to need to make requests to some REST apis, need some tests, need a task runner to manage builds and use the latest es6 and es7 features. So you sit at your terminal and type the following:

```
npm init
npm install --save express request mocha babel grunt
```

Now on my corporate network with 88.51Mbps download, 11.70Mbps upload and 21ms ping connection, that second command took 40.196 seconds to run.

Let's have a look at our working directory:

```
du -h --max-depth 1             
24M     ./node_modules
24M     .
```
```
ls -al node_modules | wc -l
198
```
```
find . -type f | wc -l
3952
```

That's right, without writing a single line of code we have 24MB and 3952 files spread over 198 modules maintained individually.

# Use compact base docker images

The official node image is based on debian jessie, containers are typically designed to run a single process so don't nececarily need a fully featured operating system. Node also provides images based on the lightweight alpine linux distribution. This image is nearly **13 times smaller** than the official debian one! When you're running regular builds that need to pull a fresh image, that can save you a hell of a lot of time.

```
docker images
REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
node                               6.9.2               178934e73268        6 days ago          651.2 MB
node                               6.9.2-alpine        de529845c111        6 days ago          50.69 MB
```

# Use the docker cache

You want to use the docker cache on your CI slaves/build agents to your advantage. With node, running `npm install` is very costly in terms of time, it downloads many files and might even compile native code which is also resource intensive. A naive Dockerfile may look like this:

```
FROM node:6.9.2-alpine
ADD . /app
RUN npm install
```

This will cause your agents to run npm install every time you check in a line of code. A smarter approach might look like this:

```
FROM node:6.9.2-alpine
RUN mkdir /app
ADD ./package.json /app/package.json
RUN npm install
ADD . /app
```

The docker caching mechanism records a hash of the file or file in each ADD directive and will only run it again if the hash differs from the one in its cache. Therefore unless you change the content of package.json, the npm install command will be served from cache, saving you precious minutes of build time.


# Keep your production images lean

To facilitate fast deployments, it's a good idea to keep the images you deploy to production as lean as possible. One strategy is to use a "build" container which contains all the tools needed to compile and test your code. The output of this container should be a tarball of your built application which can then be extracted into a much leaner "runtime" container.

# Use the timeout option for stopping containers

By default running docker stop will send a signal to interupt your app, then wait 10 seconds before it kills it. Often during build processes, we don't care about graceful shutdown, so you can set a much smaller or 0 timeout.

# Don't bloat your agents or slaves with software

Things start out like this:

Team A wants to use the brand new CI set up. Their app is pretty simple, it needs to pull their code from git, install some npm packages, run some tests using node 4 and deploy to google app engine. Team B then sees that Team A are very happy with their slick new CI / CD set up and gets their project set up. They are using python 2.7, they call out to image magick to do some image processing, then upload their app via FTP to a server running internally somewhere. Team C has been requested to build a mobile app on top of this back end, they want to use the CI set up too. Since this is a brand new app, they decide to build it with react, they start building their app and then hit a problem when running their build. They require the very latest node js version 7.10.0 to be installed which conflicts with Team A's node js version. They then request nvm to be installed so they can switch to use a different version at build time.

# Run builds for each pull request and display status

# Keep configuration in source control
# Use yarn
# Run your tests in parallel
