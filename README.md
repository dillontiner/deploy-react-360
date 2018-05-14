# Deploying Your React360 App: init to AWS in 15 Minutes

These instructions step through the process of deploying a virtual reality environment or 360 video application on AWS starting from the React360 command line interface. This tutorial assumes you have an AWS account, Docker, and Node set up. See the corresponding [blog post](https://medium.com/@dillontiner/deploying-your-reactvr-app-init-to-aws-in-15-minutes-10e77b80bbd7) for more details.

First, install the React360 CLI and create a new project.

```
$ npm install -g react-360-cli
$ react-360 init react-360-hello
$ cd react-360-hello

## Optional: run and view the project on localhost:8081/index.html
$ npm start
```

You can make edits 

### Bundle and Structure the Assets for Deployment

This follows the suggestions from the [documentation](https://facebook.github.io/react-360/docs/publish.html) on publishing a project.
```
$ npm run bundle
$ mkdir docker
$ cp build/*.js index.html docker
$ cp -r static_assets docker
```

Open up your `docker/index.html` and make the following changes
- Change `<script src="./client.bundle?platform=vr"></script>` to `<script src="./client.bundle.js?platform=vr"></script>`
- Change `'index.bundle?platform=vr&dev=true',` to `'index.bundle.js?platform=vr',`

```
## Optional: run and view the build results on localhost:8081
$ npm install -g http-server
$ cd docker
$ http-server -p 8081
```

### Create the Docker Image
Create a `Dockerfile` in the docker folder with the following as its contents:
```
FROM node:carbon
COPY ./ /
RUN npm install -g http-server
EXPOSE 8081
CMD ["http-server", "-p", "8081"]
```

Optionally build and run the image to check it before 
```
## Optional: run and view the docker-hosted project on localhost:8081
## Recommended: add these as npm scripts to streamline your workflow, see package.json for clarification
$ docker build -t react-360-hello .
$ docker run -p 8081:8081 react-360-hello
```

### Prepare the Image for AWS

Create a file called `Dockerrun.aws.json` with the following as its contents to expose port 8081:
```
{
  "AWSEBDockerrunVersion": "1",
  "Ports": [
    {
      "ContainerPort": "8081"
    }
  ]
}
```

From the docker folder, zip the project to upload to AWS. Make sure the contents are not wrapped in the parent directory when you run `unzip` on this file. This will waste your time with failed deployments if not done properly. I'd recommend using the terminal command rather than any UI unzip features as the latter might create a parent directory, which initially confused the process for me.
```
## Assumes docker folder as current directory
$ zip -r ../docker.zip ./*

## Verify zip file contents
$ mkdir ../tmp
$ cd ../tmp
$ cp ../docker.zip ./
$ unzip docker.zip
$ ls
```

This should display the same contents as running `ls` in the `docker/` folder.

### Deploy with Elastic Beanstalk

Now we just follow the AWS steps for [deploying](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/docker-singlecontainer-deploy.html) a single Docker container to Elastic Beanstalk. I've listed these here with the necessary details specified.
1. Open the Elastic Beanstalk console with this [preconfigured link](https://console.aws.amazon.com/elasticbeanstalk/home#/newApplication?applicationName=tutorials&environmentType=LoadBalanced)
2. For Platform, choose the Generic Docker option.
3. For App code, choose Upload.
4. Choose Local file, choose Browse, and open your docker.zip file
5. Choose Upload.
6. Choose Review and launch.
7. Review the available settings and choose Create app.
