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

# Use yarn
# Run your tests in parallel
