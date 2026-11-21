# Docker linter decorator

This decorator is a linter that checks whether your Dockerfile follows best practices.

## How it works

This linter downloads [dockerfilelint](http://www.dockerfilelint.com/) and searches the repository for Dockerfiles. It then scans them and displays the results in the logs.
