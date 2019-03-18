# Facebook Messenger Analysis w/ GPT-2

Original README.md for this repositories base is available at [GPT2-README.md](GPT2-README.md)

## Local Notebook

```bash
jupyter-notebook messenger-analysis.ipynb
```

## AWS SageMaker Generate

Create the SageMaker role that we'll attach to our SageMaker instance. Unfortunately since CloudFormation options for SageMaker do not allow us to attach Git repos as options yet.

```bash
aws cloudformation create-stack \
    --stack-name "fb-msg-gpt2-sagemaker-role" \
    --template-body file://cloudformation/sagemaker_role.yaml \
    --parameters ParameterKey=S3BucketName,ParameterValue=devopstar \
    --capabilities CAPABILITY_IAM
```

Once the role has been created successfully, retrieve the ARN for the use in the steps to follow.

```bash
aws cloudformation describe-stacks --stack-name "fb-msg-gpt2-sagemaker-role" \
    --query 'Stacks[0].Outputs[?OutputKey==`MLNotebookExecutionRole`].OutputValue' \
    --output text
```

It will look something like `arn:aws:iam::XXXXXXXXXXXX:role/fb-msg-gpt2-sagemaker-role-ExecutionRole-PZL3SA3IZPSN`.

Next create a Code repository and pass it in the repo [https://github.com/t04glovern/fbmsg-analysis-gpt-2](https://github.com/t04glovern/fbmsg-analysis-gpt-2)

```bash
aws sagemaker create-code-repository \
    --code-repository-name "t04glovern-gpt-2" \
    --git-config '{"Branch":"master", "RepositoryUrl" : "https://github.com/t04glovern/fbmsg-analysis-gpt-2" }'
```

Finally create the notebook instance ensuring you pass in the Role ARN from before, and the default code repository we just created.

```bash
aws sagemaker create-notebook-instance \
    --notebook-instance-name "fbmsg-gpt-2" \
    --instance-type "ml.p2.xlarge" \
    --role-arn "arn:aws:iam::XXXXXXXXXXXXX:role/fb-msg-gpt2-sagemaker-role-ExecutionRole-PZL3SA3IZPSN" \
    --default-code-repository "t04glovern-gpt-2"
```

## AWS SageMaker Delete

```bash
aws sagemaker delete-notebook-instance \
    --notebook-instance-name "fbmsg-gpt-2"

aws sagemaker delete-code-repository \
    --code-repository-name "t04glovern-gpt-2"

aws cloudformation delete-stack \
    --stack-name "fb-msg-gpt2-sagemaker-role"
```

## Attribution

- [Generating Fake Conversations by fine-tuning OpenAI's GPT-2 on data from Facebook Messenger](https://svilentodorov.xyz/blog/gpt-finetune)
- [Code for the paper "Language Models are Unsupervised Multitask Learners"](https://github.com/openai/gpt-2)
