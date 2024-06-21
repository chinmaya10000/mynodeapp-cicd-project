#### This project is for the GitLab CI/CD course

##### Test
The project uses jest library for tests. (see "test" script in package.json)
There is 1 test (server.test.js) in the project that checks whether the main index.html file exists in the project. 

To run the nodejs test:

    npm run test

Make sure to download jest library before running test, otherwise jest command defined in package.json won't be found.

    npm install

In order to see failing test, remove index.html or rename it and run tests.


Workflow Rules:
The pipeline skips execution if the branch is not main and the pipeline is not triggered by a merge request event. Otherwise, it always runs, ensuring relevant pipelines are executed as needed.

Key Variables
We set crucial variables such as IMAGE_NAME for the Docker image name and specific server hosts and endpoints for development, staging, and production environments.

Pipeline Stages
The pipeline includes several stages: Test, Build, Deploy Dev, Deploy Staging, and Deploy Prod. Each stage has specific jobs that ensure a smooth and automated workflow.

Job Highlights

run_unit_tests: This job uses a Node.js image to install dependencies and run tests. It caches the node_modules directory to speed up subsequent runs and stores test results as artifacts. This ensures that our codebase is thoroughly tested before moving forward.

sast: This job performs static application security testing (SAST) to identify vulnerabilities in the code. Running SAST helps maintain code security and quality by catching potential security issues early in the pipeline.

build_image: This job builds a Docker image and tags it with the version from package.json combined with the pipeline ID. It saves the version to a version-file.txt, providing a unique identifier for the image. This step is crucial for maintaining version control over our Docker images.

push_image: This job pushes the built Docker image to the registry. By pushing the image, we ensure that it is available for deployment in different environments. This step relies on the version saved in version-file.txt to ensure consistency.

.deploy: The .deploy job is a template for deployment jobs. It transfers docker-compose.yaml to the target server, logs into the Docker registry, and uses docker-compose to deploy the application. This reusable template streamlines the deployment process across various environments.

deploy_to_dev: This job extends the .deploy template specifically for the development environment. It sets the necessary variables for the development server and deploys the application, ensuring that the latest changes are available for testing.

run_functional_tests: After deploying to the development environment, this job runs functional tests to verify that the application works as expected. This step is crucial for validating new features and changes in a real-world scenario.

deploy_to_staging: This job extends the .deploy template for the staging environment. It sets the necessary variables for the staging server and deploys the application, providing a near-production environment for further testing and validation.

run_performance_tests: After deploying to the staging environment, this job runs performance tests to ensure the application can handle the expected load. This step helps identify and resolve performance issues before the application reaches production.

deploy_to_prod: This job extends the .deploy template for the production environment. It is triggered manually to provide control over production releases, ensuring that only approved changes are deployed to the live environment.

