# Tips for running faster builds with nodejs and docker

# Avoid npm bloat

Ok so you've got a killer idea and you're going a mind blowing web application, so you do a bit of reading and decide to use node and docker. You've done a bit of reading and are finally ready to get started. You know your app is going to need: a web framework, it's going to need to make requests to some rest apis, need some tests, need a task runner to manage builds and use the latest es6 and es7 features. So you sit at your terminal and type the following:

```
npm init
npm install --save express request mocha babel grunt
```

Now on my corporate network with 88.51Mbps download, 11.70Mbps upload and 21ms ping connection, that second comand took 40.196 seconds to run.

Let's have a look at your working directory:

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

# Use compact docker images

# Use the docker cache

# Keep your production images compact