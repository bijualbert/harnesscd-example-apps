# Git Clone Functionality for Run Step and Shell Script Step

## Purpose of the Pipeline

This pipeline demonstrates how to perform Git clone operations within Run steps and Shell Script steps in Harness pipelines, showcasing:

1. Manual Git repository cloning using authentication tokens
2. Accessing private repositories with secure token-based authentication
3. Navigating cloned repositories and performing file operations
4. Working with repositories when `cloneCodebase: false` is set
5. Compatibility with both Run steps and Shell Script steps
6. Flexibility to use in CI, CD, or Custom stages

## Pipeline Stages Overview

### Stage: s1 (CI Stage)

- **Purpose**: 
  - Demonstrates manual Git clone functionality within Run steps and Shell Script steps
  - Shows how to access and work with cloned repository contents
  - **Note**: This functionality can be implemented in CI, CD, or Custom stages

- **Key Features**:
  - `cloneCodebase: false` - Disables automatic codebase cloning
  - Caching enabled for improved performance
  - Build intelligence enabled for optimization
  - Cloud-based runtime execution
  - **Stage Flexibility**: Can be adapted for CI, CD, or Custom stage types

- **Key Steps**:

  1. **Run_1 (Run Step / Shell Script Step)**:
     - **Shell**: Sh (Bash shell execution with `set -euo pipefail` for strict error handling)
     - **Step Type**: Compatible with both Run steps and Shell Script steps
     - **Command Operations**:
       - Validates that the Git token secret is configured and non-empty
       - Validates that the repository URL is set
       - Performs Git clone using HTTPS with token authentication, with descriptive error messages on failure
       - Uses Harness secrets for secure token management
       - Verifies the cloned directory exists before navigating into it
       - Verifies the target file exists before reading it
       - Reads and displays repository contents (README.md)

## Key Configuration Elements

- **Authentication**: 
  - Uses `<+secrets.getValue("your-git-token")>` for secure Git token retrieval
  - HTTPS-based Git clone with embedded authentication

- **Repository Access**:
  - Target repository: Configured via `REPO_URL` variable (default: `https://github.com/your-org/your-repo.git`)
  - Directory navigation: Configured via `TARGET_DIR` variable (must match the cloned repository name)
  - File operations: Configured via `FILE_TO_READ` variable (default: `README.md`)

- **Platform Configuration**:
  - OS: Linux
  - Architecture: Amd64
  - Runtime: Cloud-based execution

## Use Cases

This pipeline is ideal for scenarios where:

- You need to clone multiple repositories in a single pipeline
- You want to clone repositories different from the pipeline's source repository
- You need to perform custom Git operations not available through standard codebase cloning
- You're working with private repositories requiring specific authentication methods
- You need to clone repositories conditionally based on pipeline logic
- You prefer using Shell Script steps over Run steps for Git operations

## Error Handling and Input Validation

The pipeline includes several layers of error handling and input validation to ensure reliable execution:

### Shell Strict Mode

The script begins with `set -euo pipefail` to enable strict error handling:
- `-e`: Exit immediately if any command fails
- `-u`: Treat unset variables as errors
- `-o pipefail`: Return the exit code of the first failed command in a pipeline

### Input Validation

Before performing any operations, the pipeline validates:

1. **Git Token**: Checks that the Harness secret `your-git-token` resolves to a non-empty value. If the secret is missing or empty, the step fails immediately with a clear error message and remediation guidance.
2. **Repository URL**: Validates that the repository URL is set before attempting the clone.

### Operation Error Handling

Each operation includes explicit error checking with descriptive messages:

1. **Git Clone**: If the clone fails, the error message lists possible causes (invalid token, incorrect URL, insufficient permissions, network issues) to help with quick diagnosis.
2. **Directory Navigation**: After cloning, the script verifies the expected target directory exists before attempting to `cd` into it. If the directory is missing, it lists the current directory contents to aid debugging.
3. **File Access**: Before reading a file, the script checks that the file exists. If not found, it lists available files so the user can identify the correct filename.

### Configurable Variables

The script uses clearly named variables at the top for easy customization:
- `REPO_URL`: The repository to clone
- `TARGET_DIR`: The expected directory name after cloning (should match the repository name)
- `FILE_TO_READ`: The file to display from the cloned repository

## Configuration Changes Required

### Pipeline File Location:
- **Pipeline Configuration**: `./pipeline.yaml`

### Modifications Needed in pipeline.yaml:

1. **Project and Organization Settings**:
   - Update `projectIdentifier: your-project` to match your Harness project
   - Update `orgIdentifier: default` to match your organization identifier

2. **Secret Configuration**:
   - Create a Harness secret named `your-git-token` containing your Git personal access token
   - Ensure the token has appropriate repository access permissions

3. **Repository Details**:
   - Update `REPO_URL` with your actual repository URL
   - Update `TARGET_DIR` to match the name of the directory created by cloning (typically the repository name)
   - Update `FILE_TO_READ` to the file you want to display from the cloned repository

4. **Authentication Requirements**:
   - For GitHub: Create a Personal Access Token with repo permissions
   - For GitLab: Create a Project Access Token or Personal Access Token
   - For Bitbucket: Create an App Password with repository read permissions

### Security Best Practices:

- Store Git tokens as Harness secrets, never hardcode them
- Use tokens with minimal required permissions
- Regularly rotate authentication tokens
- Consider using SSH keys for enhanced security in production environments

## Troubleshooting

| Symptom | Possible Cause | Resolution |
|---------|---------------|------------|
| `ERROR: Git token secret 'your-git-token' is empty or not configured.` | Harness secret not created or has an empty value | Create a Harness secret named `your-git-token` with a valid Git personal access token |
| `ERROR: Failed to clone repository` with `fatal: Authentication failed` | Token is invalid, expired, or lacks permissions | Regenerate the token and ensure it has `repo` (read) scope |
| `ERROR: Failed to clone repository` with `fatal: repository not found` | Incorrect repository URL or the token doesn't have access to the repository | Verify the URL and confirm the token owner has access to the repository |
| `ERROR: Expected directory '...' not found after clone.` | `TARGET_DIR` variable doesn't match the cloned repository's directory name | Set `TARGET_DIR` to the repository name (the last path segment of the repo URL without `.git`) |
| `ERROR: File '...' not found in '...'` | The specified file doesn't exist in the repository | Check the repository contents and update `FILE_TO_READ` to a valid filename |
| Clone hangs or times out | Network connectivity issues or firewall blocking HTTPS Git traffic | Verify network access to the Git provider from the build environment |

## Example Scenarios

- **Multi-repository workflows**: Clone additional repositories for dependencies or configuration files
- **Cross-repository operations**: Access files from different repositories within the same pipeline
- **Custom Git workflows**: Implement specific Git operations like sparse checkouts or shallow clones
- **Repository mirroring**: Clone repositories for backup or synchronization purposes
