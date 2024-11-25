pipeline {
    agent none
    parameters {
        choice(name: 'edge', choices: ['edgemorocco005', 'edgejordan001', 'edgeuk006'], description: 'Choose an edge')
        string(name: 'date', defaultValue: '', description: 'Default=Today, Date in MM/DD/YYYY format') // Placeholder
        string(name: 'start_time', defaultValue: "14:00", description: 'Start time in HH:MM format (e.g., 14:00)')
        string(name: 'end_time', defaultValue: "14:30", description: 'End time in HH:MM format (e.g., 14:30)')
        booleanParam(name: 'hdf5 files', defaultValue: true )
        booleanParam(name: 'npz files', defaultValue: false )
        booleanParam(name: 'Collect_config_file', defaultValue: true )        
        string(name: 'temperature', defaultValue: "12C" , description: 'Temperature')
        string(name: 'desc', defaultValue:"Leak Detected", description: 'Notes:')
        text defaultValue: '''
{
    "edgeConfig": {
        "ipAddress": "192.168.8.100",
        "build": "Production",
        "serialNumber": "2407-0736-0002"
    }
}
''', description: 'Example of Json input', name: 'multiline_json'
    
    }
    environment {
        TODAY_DATE = new Date().format('MM/dd/yyyy') // Use Groovy to calculate today's date
    }
    stages {
        stage('Validate Parameters') {
            agent { label 'main' }
            steps {
                script {
                    if (!params.date?.trim()) {
                        env.date = env.TODAY_DATE // Assign default value dynamically
                        echo "Date parameter was empty; defaulted to ${env.date}"
                    } else {
                        echo "Date parameter provided: ${params.date}"
                    }
                    
                    // Regular expressions for validation
                    def timeRegex = /^[0-2][0-9]:[0-5][0-9]$/
                    def dateRegex = /^(0[1-9]|1[0-2])\/(0[1-9]|[1-2][0-9]|3[0-1])\/\d{4}$/
                    
                    // Validate start_time and end_time format
                    if (!params.start_time.matches(timeRegex)) {
                        error "Invalid start_time format. Expected HH:MM (e.g., 14:00). Got: ${params.start_time}"
                    }
                    if (!params.end_time.matches(timeRegex)) {
                        error "Invalid end_time format. Expected HH:MM (e.g., 14:30). Got: ${params.end_time}"
                    }

                    // Validate date format
                    if (!env.date.matches(dateRegex)) {
                        error "Invalid date format. Expected MM/DD/YYYY. Got: ${env.date}"
                    }

                    // Parse start_time and end_time
                    def startSplit = params.start_time.split(':')
                    def startHours = startSplit[0].toInteger()
                    def startMinutes = startSplit[1].toInteger()
                    def endSplit = params.end_time.split(':')
                    def endHours = endSplit[0].toInteger()
                    def endMinutes = endSplit[1].toInteger()

                    // Convert to minutes since midnight
                    def startTimeInMinutes = startHours * 60 + startMinutes
                    def endTimeInMinutes = endHours * 60 + endMinutes

                    // Compare times
                    if (endTimeInMinutes <= startTimeInMinutes) {
                        error "end_time (${params.end_time}) must be greater than start_time (${params.start_time})."
                    }
                }
            }
        }
        stage('Job Preperation') {
            agent { label 'main' }
            steps {
                script {
                    sh """
                    S3_BUCKET="lwf-dw-storage"
                    S3_PATH="s3://$S3_BUCKET/hdf5-files/"
                    EXPIRATION="86400"  # 24 hours in seconds
                    
                    echo "Generating a 24-hour download link..."
                    DOWNLOAD_URL=\$(aws s3 presign "\${S3_PATH}" --expires-in \$EXPIRATION)
                    """
                }
            }
        }
        stage('Collect Data & Upload from Edge') {
            agent { label 'edgeil001' }
            steps {
                script {
                   writeFile file: 'timestamp_logger.sh', text: '''#!/bin/bash                    
echo "Stage 2, Step 1: hello ${env.TODAY_DATE}"

# Define the output file
output_file="timestamps.log"
count=0
# Loop 10 times
while [ $count -lt 10 ]; do
  # Get the current timestamp
  timestamp=$(date +"%Y-%m-%d %H:%M:%S")
  
  # Print the timestamp to the terminal
  echo "$timestamp"
  echo "$count"
  # Append the timestamp to the file
  echo "$timestamp" >> "$output_file"
  # Increment the counter
  count=$((count + 1))
  # Wait for 10 seconds
  sleep 10
done

echo "Done! Timestamps written to $output_file"
                    '''
                    
                    // Make the script executable
                    sh 'chmod +x timestamp_logger.sh'

                    // Run the script
                    sh './timestamp_logger.sh'                    
                }
                sh 'echo "Stage 2, Step 2: hello"'
            }
        }
        stage('Update DataWarehouse DB') {
            agent { label 'main' }
            steps {
                sh 'echo "Stage 3, Step 1: hello"'
                sh 'echo "Stage 3, Step 2: hello"'
            }
        }
        stage('Send Notification') {
            agent { label 'main' }
            steps {
                sh 'echo "Stage 3, Step 1: hello"'
                sh 'echo "Stage 3, Step 2: hello"'
            }
        }
        
    }
}
