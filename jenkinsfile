pipeline {
    agent any

    // === CRON SCHEDULES ===
    // Jenkins runs in UTC
    // 9:00 AM IST  = 03:30 UTC
    // 9:00 PM IST  = 15:30 UTC
    triggers {
        cron('30 3 * * *')     // 9:00 AM IST
        cron('30 15 * * *')    // 9:00 PM IST
    }

    stages {
        stage('Manage AWS EC2 Based on IST Time') {
            steps {
                script {
                    sh '''
                    echo "Jenkins Pipeline Running...."
                    echo "Checking IST Time Logic"

                    # Get current IST time
                    current_time=$(TZ=Asia/Kolkata date +%H:%M)
                    echo "Current IST Time: $current_time"

                    hour=$(echo "$current_time" | cut -d':' -f1)
                    minute=$(echo "$current_time" | cut -d':' -f2)

                    # Convert to minutes since midnight
                    total_minutes=$((10#$hour * 60 + 10#$minute))

                    # Decide action
                    if [[ "$total_minutes" -ge 540 && "$total_minutes" -lt 1260 ]]; then
                      # Between 09:00 (540) and 21:00 (1259)
                      action="start"
                    else
                      action="stop"
                    fi

                    echo "Final Action: $action"

                    INSTANCE_ID="i-02b8041d7c0ccf9ae"

                    current_status=$(aws ec2 describe-instances --instance-ids $INSTANCE_ID --query "Reservations[].Instances[].State.Name" --output text)
                    echo "Current Instance State: $current_status"

                    if [[ "$action" == "start" ]]; then
                      if [[ "$current_status" == "running" ]]; then
                        echo "Instance already running. No action needed."
                      else
                        echo "Starting Instance..."
                        aws ec2 start-instances --instance-ids $INSTANCE_ID
                      fi
                    fi

                    if [[ "$action" == "stop" ]]; then
                      if [[ "$current_status" == "stopped" ]]; then
                        echo "Instance already stopped. No action needed."
                      else
                        echo "Stopping Instance..."
                        aws ec2 stop-instances --instance-ids $INSTANCE_ID
                      fi
                    fi
                    '''
                }
            }
        }
    }
}
