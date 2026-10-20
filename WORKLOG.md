# Threatlocker Technical Assessment

#### Author: Aryss Fennell
#### Github: aegomez1

#### Environment:
- OS: macOS
- Tools: .NET 8 SDK, Docker, Trivy, Github Actions

## Part 1 - Scaffold a real service
- Decided to delete example template code since it's not needed.
- Endpoints displayed correctly, override works.

## Part 2 - Vulnerability scanning
- Problem: Trivy encountered a high, critical vulnerability with a fix, named perl-base, in the image.
- Actions: Added `RUN apt-get update && apt-get install -y --only-upgrade perl-base && rm -rf /var/lib/apt/lists/*` during runtime in the Dockerfile.
- Decisions: Thought about previous command to update vulnerability in dotnet, but this was in the image. Decided to put command in the Dockerfile, so it can update only perl-base, as image is being created layer by layer. Deleted update list to reduce unnecessary files.
- Result: Ran Trivy again and results came back clean.

- Answer: The container runs as root by default, which gives it host-level privileges since containers share the host-kernal. After performing privileged operations, the user needs to switch to a non-root user to give it the least amount of privileges and access to the host.

## Part 3 - Github Actions
- Problem 1: Workflow #1 failed because of a syntax error in `dotnet list package` of format-and-test.
- Action/Decision: Looked at workflow log file and discovered it was a syntax error, so I fixed it.
- Results: Workflow #2 ran successfully.
- Problem 2: Workflow #3 failed because of another syntax error on line 37 of docker-build-and-scan.
- Action/Decision: Looked at workflow log file and discoverd a syntax error, so I fixed it.
- Problem 3: Workflow #4 failed, looked at log file and ci.yml and realized I forgot to add the build-push image step on ci.yml.
- Result: Workflow #5 ran successfully.
- Problem 4: Workflow #7 failed, looked at log file and realized I had an expression error on line 64. I forgot to put a closing bracket on `${{ github.actor }` .
- Result: Workflow #8 ran successfully.
- Thoughts: Was looking over docker-build-and-scan and push jobs, and saw the image was being built twice- Why not merge the two jobs since they're not running in parallel and it'll save some time from not building the image twice.
- Results: Workflow #9 failed but workflow #10 ran successfully after fixing error, which was forgetting to add `${{ env.REGISTRY }}` in image-ref..
- Problem 5: Workflow #11 failed with the workflow log saying there was an error on line 64.
- Actions/Decisions: I couldn't find anything wrong with line 64 that just says `steps:` , but I fixed an expression in deploy. Workflow #12 still failed. After much scanning, I discovered I forgot a closing bracket in line 65..

## Part 4 - Deploy step
- Ran commands through my terminal. Had to make the orderservice package public to be able to call it from my terminal without a password. Decided to also run it through the pipeline.
```bash
docker pull ghcr.io/aegomez1/orderservice:f24990d314bee8b021c68bb07963ed46c371a895
docker stop orderservice-prod 2>/dev/null; docker rm orderservice-prod 2>/dev/null
docker ps
docker run -d --name orderservice-prod -p 8080:8080 -e APP_VERSION=f24990d ghcr.io/aegomez1/orderservice:f24990d314bee8b021c68bb07963ed46c371a895
docker ps
curl http://localhost:8080/version
curl http://localhost:8080/health
```

- Problem 6: Workflow #14 failed because I put the wrong job name for deploy's dependency- fixed it. 
- Problem 7: Workflow #15 failed because no image was found to stop after running gate : `docker stop orderservice-prod; docker rm orderservice-prod || true` 
- Actions/Decisions: Docker stops because it couldn't find a container to stop but docker rm would still continue because of the `|| true` expression even if there was nothing to remove. It worked in the terminal but not through the pipeline. I'll add `|| true` to the end of the docker stop expression so it'll continue running even if it finds no container to stop.
- Results: Workflow #16 failed but my fixed worked.
- Problem 8: Workflow #16 failed with error: `curl: (56) Recv failure: Connection reset by peer`.
- Actions/Decisions: Decided to add `run docker ps` , `run docker logs orderservice-prod` , and `run curl --fail http://localhost:8080/health` so I could see what was happening to the container and check if it was healthy.
- Result: Discovered curl commands were running while container was still starting, so decided to put a sleep 3 timer to give it time to spin up. It fixed the problem and now the pipeline runs end to end.
- Answer: 

## Part 5: Written section
