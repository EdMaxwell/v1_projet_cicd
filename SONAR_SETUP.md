# SonarCloud Setup Guide for BobApp

This guide explains how to set up and test the SonarCloud integration for the first time.

## Prerequisites

Before running the CI pipeline with SonarCloud integration, you need to:

### 1. Create a SonarCloud Organization

1. Go to [sonarcloud.io](https://sonarcloud.io)
2. Sign in with your GitHub account
3. Click on "+" → "Create new organization"
4. Choose "edmaxwell" as the organization key (must match `sonar.organization` in `sonar-project.properties`)
5. Select the free plan for public projects

### 2. Import the Project

1. In SonarCloud, go to "+" → "Analyze new project"
2. Select the repository: `EdMaxwell/v1_projet_cicd`
3. Click "Set Up"
4. Choose "With GitHub Actions" as the analysis method
5. SonarCloud will generate a `SONAR_TOKEN` for you

### 3. Disable Automatic Analysis

**Important**: You must disable Automatic Analysis to avoid conflicts with CI analysis.

1. Go to your project in SonarCloud
2. Navigate to **Administration → Analysis Method**
3. Turn **OFF** the "Automatic Analysis" toggle
4. Save the changes

This is required because running both CI analysis and Automatic Analysis simultaneously will cause the analysis to fail.

### 4. Add GitHub Secret

1. Go to your GitHub repository settings
2. Navigate to **Settings → Secrets and variables → Actions**
3. Click **New repository secret**
4. Name: `SONAR_TOKEN`
5. Value: Paste the token from SonarCloud
6. Click **Add secret**

## Configuration Files

The following files have been created/modified:

### `sonar-project.properties` (root)
- Main configuration file for SonarCloud
- Defines project key, organization, sources, tests, and coverage paths
- Configured for mono-repo structure (backend + frontend)

### `.github/workflows/ci.yml`
- New job `sonar` added after `backend` and `frontend` jobs
- Downloads coverage artifacts (JaCoCo + LCOV)
- Runs SonarCloud analysis
- Checks Quality Gate status

### `docs/step-03-sonar.md`
- Comprehensive documentation explaining the Sonar integration
- Includes configuration details, Quality Gate proposals, and troubleshooting

## Testing the Integration

### Option 1: Test on a PR (Recommended)

1. Create a test branch:
   ```bash
   git checkout -b test/sonar-integration
   git push -u origin test/sonar-integration
   ```

2. Create a PR to `main` or `develop`

3. The CI pipeline will run automatically with 4 jobs:
   - `backend` - Build and test backend
   - `frontend` - Build and test frontend  
   - `summary` - Generate overall summary
   - `sonar` - Analyze code quality

4. Check the GitHub Actions tab to see the job status

5. Review the SonarCloud dashboard:
   - Go to https://sonarcloud.io/dashboard?id=EdMaxwell_v1_projet_cicd
   - Verify that both backend and frontend code are analyzed
   - Check coverage metrics

### Option 2: Test on a Direct Push

1. Push to `main` branch (will trigger the workflow)

2. Monitor the workflow in GitHub Actions

3. Review SonarCloud results

## Verification Checklist

After the first successful run, verify:

- [ ] ✅ All 4 jobs completed (backend, frontend, summary, sonar)
- [ ] ✅ SonarCloud job shows "Analysis successful"
- [ ] ✅ Coverage files were downloaded (check logs of sonar job)
- [ ] ✅ SonarCloud dashboard shows the project
- [ ] ✅ Both Java and TypeScript code are analyzed
- [ ] ✅ Coverage metrics are displayed (not 0%)
- [ ] ✅ Quality Gate status is visible

## Troubleshooting

### Issue: SONAR_TOKEN not found

**Solution**: Make sure the `SONAR_TOKEN` secret is configured in GitHub repository settings.

### Issue: Coverage is 0% in SonarCloud

**Possible causes**:
1. Coverage artifacts not generated/uploaded in backend/frontend jobs
2. Coverage file paths in `sonar-project.properties` don't match actual files
3. Coverage artifacts not downloaded correctly

**Solution**: 
- Check the "Verify coverage files" step in the sonar job logs
- Verify that backend and frontend jobs uploaded artifacts successfully
- Download artifacts manually from GitHub Actions to check the file structure

### Issue: SonarCloud shows "Project not found"

**Solution**: 
- Verify that the project was imported in SonarCloud
- Check that `sonar.projectKey` and `sonar.organization` in `sonar-project.properties` match exactly what's configured in SonarCloud

### Issue: Quality Gate fails

This is expected on the first run if the code has quality issues. You can:
1. Review the issues in SonarCloud dashboard
2. Set `continue-on-error: true` in the Quality Gate step (already configured)
3. Fix critical issues progressively
4. Adjust Quality Gate thresholds in SonarCloud settings

## Next Steps

After successful integration:

1. **Review initial results**: Check the SonarCloud dashboard for bugs, vulnerabilities, and code smells
2. **Configure Quality Gate**: Adjust thresholds based on the recommendations in `docs/step-03-sonar.md`
3. **Fix critical issues**: Prioritize security vulnerabilities and critical bugs
4. **Team training**: Familiarize the team with SonarCloud dashboard and reports
5. **Continuous improvement**: Gradually increase coverage and quality thresholds

## Additional Resources

- [SonarCloud Documentation](https://docs.sonarcloud.io/)
- [SonarCloud GitHub Action](https://github.com/SonarSource/sonarcloud-github-action)
- [Quality Gate Documentation](https://docs.sonarcloud.io/improving/quality-gates/)
- Project Documentation: `docs/step-03-sonar.md`
