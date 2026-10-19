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
