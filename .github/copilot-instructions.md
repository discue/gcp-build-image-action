# GCP Build Image Action

GCP Build Image Action is a GitHub Action that builds and pushes container images to Google Cloud using Cloud Build with Cloud Native Buildpacks. This is a composite action written in YAML that wraps `gcloud builds submit` commands.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Repository Structure
- `action.yml` - Main GitHub Action definition (composite action)
- `README.md` - Documentation with usage examples and API reference
- `.github/workflows/` - Contains test and example workflows
- `example/` - Simple Node.js Express app for testing the action
- No traditional build process (this is a GitHub Action, not a software project)

### Validation and Testing
- Validate action syntax: `python3 -c "import yaml; yaml.safe_load(open('action.yml'))"`
- Check required fields: `grep -q "name:\|description:\|runs:" action.yml`
- Validate workflows: `python3 -c "import yaml; yaml.safe_load(open('.github/workflows/test.yml'))"`
- Run validation script:
  ```bash
  # Test action.yml exists and is valid YAML
  [ -f action.yml ] && python3 -c "import yaml; yaml.safe_load(open('action.yml'))" && echo "✅ action.yml syntax valid"
  
  # Check required action fields
  for field in "name:" "description:" "runs:"; do
    grep -q "$field" action.yml && echo "✅ Found $field" || (echo "❌ Missing $field" && exit 1)
  done
  
  # Check README sections
  for section in "## Usage" "## Inputs"; do
    grep -q "$section" README.md && echo "✅ Found $section" || (echo "❌ Missing $section" && exit 1)
  done
  ```

### Example Application Testing
- Install dependencies: `cd example && npm install` -- takes ~10 seconds
- Run tests: `npm test` -- takes <1 second (currently just echoes "No tests specified")
- Start server: `npm start` -- starts immediately on port 3000
- Test endpoints:
  - Main: `curl http://localhost:3000` (returns JSON with message, timestamp, version)
  - Health: `curl http://localhost:3000/health` (returns {"status":"healthy"})
- NEVER CANCEL: Server runs indefinitely until stopped with Ctrl+C

### GCP Authentication Requirements
**CRITICAL**: This action requires Google Cloud authentication to function. Without proper GCP setup:
- `gcloud auth list` will show "No credentialed accounts"
- `gcloud builds submit` commands will fail with authentication errors
- Testing is limited to syntax validation and example app functionality

**Expected GCP setup for real usage**:
- Service account with Cloud Build Editor permissions
- Artifact Registry Writer permissions
- Cloud Build API enabled
- Valid project ID and region configured

### Core Action Functionality
The action executes this command structure:
```bash
gcloud builds submit --quiet --pack image=IMAGE_URL SOURCE_DIR
```

Where:
- `IMAGE_URL` is the full container registry path (e.g., `europe-west3-docker.pkg.dev/project/repo/image`)
- `SOURCE_DIR` is the source directory to build (defaults to `.`)
- `--quiet` flag is used by default (can be disabled)
- `--pack` uses Cloud Native Buildpacks for automatic image creation

## Validation Scenarios

### ALWAYS run these validation steps after making changes:
1. **Syntax validation**: Validate all YAML files are syntactically correct
2. **Action metadata**: Ensure required fields exist in action.yml
3. **Example app testing**: Install, run, and test the example Node.js application
4. **README validation**: Verify documentation sections are present
5. **Workflow testing**: Confirm GitHub workflows are valid

### Manual validation workflow:
```bash
# 1. Validate action syntax (required)
python3 -c "import yaml; yaml.safe_load(open('action.yml'))" && echo "✅ action.yml valid"

# 2. Test example application (required)
cd example
npm install  # ~10 seconds
npm start &  # Start in background
SERVER_PID=$!
sleep 2
curl -s http://localhost:3000 | grep -q "Hello from GCP Build Image Action" && echo "✅ Server responds correctly"
curl -s http://localhost:3000/health | grep -q "healthy" && echo "✅ Health endpoint works"
kill $SERVER_PID
cd ..

# 3. Validate required fields (required)
grep -q "name:\|description:\|runs:" action.yml && echo "✅ Required fields present"

# 4. Check documentation (required)
grep -q "## Usage\|## Inputs" README.md && echo "✅ Documentation sections present"
```

## Common Tasks

### Testing Changes to action.yml
- Always validate YAML syntax first: `python3 -c "import yaml; yaml.safe_load(open('action.yml'))"`
- Check that all required fields are present
- Verify the shell script in the `run:` section is syntactically correct
- Test with the example workflow (though it won't actually build without GCP auth)

### Testing Changes to Documentation
- Verify README.md sections are complete and accurate
- Ensure usage examples match the actual action.yml inputs/outputs
- Check that prerequisite information is up to date

### Adding New Features
- Update action.yml inputs/outputs sections
- Update the shell script in the `run:` section
- Update README.md with new input/output documentation
- Update example workflow if needed
- Test with the example application

### Timing Expectations
- NEVER CANCEL: All validation steps complete in under 30 seconds
- Example app installation: ~10 seconds
- Example app startup: immediate
- YAML validation: <1 second
- README checks: <1 second
- No long-running builds (this is a GitHub Action, not a compiled project)

## Repository Information

### Key Files Output
```
ls -la
total 40
drwxr-xr-x 5 runner docker 4096 Aug  9 20:36 .
drwxr-xr-x 3 runner docker 4096 Aug  9 20:35 ..
drwxr-xr-x 7 runner docker 4096 Aug  9 20:36 .git
drwxr-xr-x 3 runner docker 4096 Aug  9 20:36 .github
-rw-r--r-- 1 runner docker 2152 Aug  9 20:36 .gitignore
-rw-r--r-- 1 runner docker 1062 Aug  9 20:36 LICENSE
-rw-r--r-- 1 runner docker 5201 Aug  9 20:36 README.md
-rw-r--r-- 1 runner docker 1808 Aug  9 20:36 action.yml
drwxr-xr-x 2 runner docker 4096 Aug  9 20:36 example
```

### Example Directory Contents
```
ls -la example/
total 16
drwxr-xr-x 2 runner docker 4096 Aug  9 20:36 .
drwxr-xr-x 5 runner docker 4096 Aug  9 20:36 ..
-rw-r--r-- 1 runner docker  362 Aug  9 20:36 package.json
-rw-r--r-- 1 runner docker  425 Aug  9 20:36 server.js
```

### Available Tools
- Node.js v20.19.4 and npm 10.8.2 (for example app)
- Python 3 (for YAML validation)
- Google Cloud SDK 532.0.0 (but no authentication configured)
- Standard bash tools for script validation

## Limitations and Constraints

### Cannot Actually Test GCP Functionality
- No GCP authentication available in development environment
- Cannot test actual Cloud Build submission
- Cannot validate image builds or registry pushes
- Testing is limited to syntax validation and example app functionality

### No Linting or Formatting Tools
- No ESLint, Prettier, or similar tools configured
- No automated code formatting available
- Manual code review required for style consistency
- Follow existing patterns in action.yml and documentation

### GitHub Action Limitations
- Cannot test action execution without pushing to GitHub and triggering workflows
- Local testing limited to syntax validation
- Real functionality testing requires GCP setup and GitHub Actions environment

## Error Prevention

### Common Mistakes to Avoid
- Do not modify the core gcloud command structure in action.yml without understanding buildpack requirements
- Do not remove required fields from action.yml (name, description, runs, inputs)
- Do not change the composite action type without major restructuring
- Always validate YAML syntax before committing changes
- Ensure README.md examples match actual action.yml interface

### Before Committing Changes
1. Run all validation commands listed above
2. Test the example application installation and execution
3. Verify YAML syntax for all workflow and action files
4. Check that documentation is consistent with code changes
5. Ensure no new files are accidentally included (check .gitignore)