# Docker linter decorator

This decorator is a linter which checks that your Dockerfile follow good practices.

## How it works

This linter download a tool [dockerfilelint](http://www.dockerfilelint.com/) and scan the repo to file a Dockerfile. It then scan the file and displays the results within the logs.
