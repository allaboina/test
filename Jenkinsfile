pipeline {
    agent any

    environment {
        REPO = "allaboina/test"
        GITHUB_USER = "allaboina"
        GITHUB_TOKEN = credentials('github-pat')  // your Jenkins stored PAT
    }

    stages {

        stage('Detect Latest PR') {
            steps {
                script {
                    echo "🔍 Detecting latest open Pull Request..."
                    def response = sh(
                        script: """
                        curl -s -H "Authorization: token ${GITHUB_TOKEN}" \
                        -H "Accept: application/vnd.github+json" \
                        https://api.github.com/repos/${REPO}/pulls?state=open \
                        | jq -r '.[0].number'
                        """,
                        returnStdout: true
                    ).trim()

                    if (response == "null" || response == "") {
                        error("No open PRs found.")
                    } else {
                        env.PR_NUMBER = response
                        echo "✅ Latest PR detected: #${env.PR_NUMBER}"
                    }
                }
            }
        }

        stage('Auto Approve and Merge PR') {
            steps {
                script {
                    echo "⚙️ Attempting PR auto-approval and merge..."

                    // Try approving PR (skip if self-owned)
                    sh """
                    response=$(curl -s -X POST \
                      -H "Authorization: token ${GITHUB_TOKEN}" \
                      -H "Accept: application/vnd.github+json" \
                      https://api.github.com/repos/${REPO}/pulls/${PR_NUMBER}/reviews \
                      -d '{"event":"APPROVE"}')

                    if echo "$response" | grep -q "Can not approve your own pull request"; then
                        echo "Skipping approval — cannot approve own PR."
                    else
                        echo "PR approved successfully."
                    fi
                    """

                    // Merge PR
                    sh """
                    curl -s -X PUT \
                      -H "Authorization: token ${GITHUB_TOKEN}" \
                      -H "Accept: application/vnd.github+json" \
                      https://api.github.com/repos/${REPO}/pulls/${PR_NUMBER}/merge \
                      -d '{"merge_method":"squash"}' | jq .
                    """
                }
            }
        }

        stage('Post-Merge Validation') {
            steps {
                script {
                    echo "🔁 Verifying merge status..."
                    sh """
                    curl -s -H "Authorization: token ${GITHUB_TOKEN}" \
                    -H "Accept: application/vnd.github+json" \
                    https://api.github.com/repos/${REPO}/pulls/${PR_NUMBER} \
                    | jq '.merged, .state'
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Auto-PR workflow completed successfully."
        }
        failure {
            echo "❌ Auto-PR workflow failed. Please review logs."
        }
    }
}

