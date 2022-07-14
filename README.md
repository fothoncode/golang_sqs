# Golang SQS Consumer

golang_sqs is a golang module that used to consume AWS SQS queue through channel

## Install

``` go
go get github.com/fothoncode/golang_sqs
```

## Example code

``` go
import (
	"fmt"
	"time"


	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/credentials"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/sqs"
    
	sqsGo "github.com/fothoncode/golang_sqs"
)

func GetQueueURL(sess *session.Session, queue string) (*sqs.GetQueueUrlOutput, error) {
	sqsClient := sqs.New(sess)

	result, err := sqsClient.GetQueueUrl(&sqs.GetQueueUrlInput{
		QueueName: &queue,
	})

	if err != nil {
		return nil, err
	}

	return result, nil
}

func handle(m *sqs.Message) error {
	/* Handler code */

	time.Sleep(time.Second * 2)
	return nil
}

func Main() {
    AWSRegion := "xxx"
    AWSAccessKeyId := "xxx"
    AWSSecretAccessKey := "xxx"
    QueueKey := "queueName"

	sess, err := session.NewSessionWithOptions(session.Options{
		Profile: "default",
		Config: aws.Config{
			Region:      aws.String(AWSRegion),
			Credentials: credentials.NewStaticCredentials(AWSAccessKeyId, AWSSecretAccessKey, ""),
		},
	})

	if err != nil {
		fmt.Printf("Failed to initialize new session: %v", err)
		return
	}

	urlRes, err := GetQueueURL(sess, QueueKey)
	if err != nil {
		fmt.Printf("Got an error while trying to get queue url: %v", err)
		return
	}
	fmt.Println("Queue URL: " + *urlRes.QueueUrl)

	c := sqsGo.New(*urlRes.QueueUrl, handle,
		&sqsGo.Config{
			AwsSession:                  sess,
			Receivers:                   1,
			SqsMaxNumberOfMessages:      10,
			SqsMessageVisibilityTimeout: 20,
			PollDelayInMilliseconds:     100,
		})

	c.Start()
}

```