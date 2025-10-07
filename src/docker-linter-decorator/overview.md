# Docker linter decorator

This decorator is a linter which checks that your Dockerfile follows good practices.

## How it works

This linter downloads a tool [dockerfilelint](http://www.dockerfilelint.com/) and scans the repo to find a Dockerfile. It then scans the file and displays the results within the logs.
