Docker containers creation:
• Create Dockerfile - defines what goes on in your containers. Add here access to resources from the 'outside world', like files of your app and requirements.
• Specify the requirements (libraries) in a separate file and include it as per the above.
• Run Build - create docker image. Worth adding a tag (descriptive name) when creating it.
• Run the app. To access it locally, map the port of your local machine to the one exposed in dockerfile.
• Share the image to registry. Registry -> Collection of repos. Repo -> Collection of images.
	- You can create a docker account to use docker public registry.
• Connect to docker account. Now whenever you run image locally, it will be pulled if not found.