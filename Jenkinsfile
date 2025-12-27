pipeline {
    agent any

    // Jenkins cron runs in UTC
    // 9:40 PM IST = 16:10 UTC
    // 9:45 PM IST = 16:15 UTC
    triggers {
        cron('''
            10 16 * * *
            15 16 * * *
        ''')
    }

    stages {

        stage("Test Pipeline Running") {
            steps {
                sh 'echo "Pipeline Successfully Triggered..."'
            }
        }

        stage('Manage AWS EC2 Based on IST Time') {
            steps {
                withAWS(credentials: 'aws-account-tripare', region: 'ap-south-1') {

                    sh '''#!/bin/bash

                    REGION="ap-south-1"
                    INSTANCE_ID="i-02b8041d7c0ccf9ae"

                    echo "=============================="
                    echo " AWS EC2 Auto Start/Stop JOB "
                    echo "=============================="

                    current_time=$(TZ=Asia/Kolkata date +%H:%M)
                    echo "Current IST Time: $current_time"

                    # ------ TESTING LOGIC ------
                    if [[ "$current_time" == "21:40" ]]; then
                      action="stop"
                    elif [[ "$current_time" == "21:45" ]]; then
                      action="start"
                    else
                      echo "Unknown time trigger, exiting safely..."
                      exit 0
                    fi
                    # ---------------------------

                    echo "Final Action: $action"

                    echo "Checking Current Instance State..."
                    current_status=$(aws ec2 describe-instances --instance-ids $INSTANCE_ID --region $REGION --query "Reservations[].Instances[].State.Name" --output text)
                    echo "Current Instance State: $current_status"

                    if [[ "$action" == "start" ]]; then
                      if [[ "$current_status" == "running" ]]; then
                        echo "Instance already running. No action needed."
                      else
                        echo "Starting EC2..."
                        aws ec2 start-instances --instance-ids $INSTANCE_ID --region $REGION
                      fi
                    fi

                    if [[ "$action" == "stop" ]]; then
                      if [[ "$current_status" == "stopped" ]]; then
                        echo "Instance already stopped. No action needed."
                      else
                        echo "Stopping EC2..."
                        aws ec2 stop-instances --instance-ids $INSTANCE_ID --region $REGION
                      fi
                    fi

                    echo "Job Completed Successfully"
                    '''
                }
            }
        }
    }
}
