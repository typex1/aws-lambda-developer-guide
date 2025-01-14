20230622

Official *exam* page: https://aws.amazon.com/certification/certified-devops-engineer-professional/?ch=sec&sec=rmg&d=1
Automatic extension of:
•	SysOps Associate
•	Dev Associate !!

75 Questions, 10 of them are unscored, passing score: 750/1000
180+30 = 210 minutes, 75 questions -> 2.8 minutes per question.

Compare to SAA:
160 minutes, 65 questions -> 2.4 minutes per question

aCloudguru course contents: approx. 60 hours!
https://learn.acloud.guru/course/aws-certified-devops-engineer-pro/dashboard

Update for DOP-C02 – in March 2023.

Official practice exam in Skillbuilder:
https://explore.skillbuilder.aws/learn/course/14959/play/73913/aws-certified-devops-engineer-professional-official-practice-exam-dop-c02-english

Exam readiness DIG (interesting: https://phonetool.amazon.com/users/aggalvin is in the same team as Morgan Willis): This is DOP-C01 though!
https://explore.skillbuilder.aws/learn/course/74/play/498/exam-readiness-aws-certified-devops-engineer-professional 

Learnings from the above DIG:
•	Domain 1: SDLC Automation
•	CodeBuild: a serverless replacement (or combined part) of a Jenkins server
o	Use buildspec file to configure
•	CodeDeploy: deployment agent installed on EC2 instances
o	Uses appspec file to configure
o	Revision = source code files + appspec file
•	CodePipeline = pipeline orchestration, butter than Lambda(s)!
o	Can integrate with 3rd party services
o	Can request human approval (via SNS topic)
•	Testing pyramid
o	Foundation: unit tests – always make sure you have a good code coverage
o	Next levels: service/integration/component, then performance/compliance
o	Also infrastructure code needs to be tested!
o	Also necessary: regression tests! -> TDD write tests before you write code!
•	Also important: fault tolerance testing! FT is a huge theme on the exam
•	Deployment types:
o	In-place, most simple one. Benefit: quick. Cons: outage
o	Rolling updates: update instances step by step
o	Blue/green deployments: 0/100 – 50/50 – 100/0
o	Red-black deployment: create new env, deploy there, in case of success remove old env
o	Immutable: create the full arch new, then delete the old one
•	Sample questions:
o	Use git-secrets (open source https://github.com/awslabs/git-secrets by AWS Labs) as a pre-commit hook in CodeCommit to detect security credentials being committed to the repo. Pre-commit hook: evaluated BEFORE the update is committed to the repo!
o	How can you go about reducing the footprint of your Jenkins build environment?
	 
•	Domain 2: Config management and IaC
o	CloudFormation
	Change sets: Be sure about update options (UpdatePolicy section)
	Updates: with interruption, without interruption, replacement
o	 
o	CloudFormation template chaining and nesting!
o	Default behaviour: CFN attempts to create resources in parallel.
o	“dependson” is an important keyword.
o	WaitConditionHandle, CreationPolicy – dive deep here!
o	Also Stack Policies
o	Custom resource workflow
•	Sample questions:
o	CFN: UPDATE_ROLLBACK_FAILED. Possible reason?
	Answer: Stack’s resources have been changed outside of CFN
o	CFN blue/green deployment: quick roll back should be possible over the next 24 hours. Answer: create new launch config, new ASG, new LB. weighted Route 53 public record and slowly transition traffic from blue to green.
o	CFN: Stack DELETE failed: a VPC to be deleted has EC2 instances which have been created outside of the template. And: The stack(s) have termination protection enabled.
o	Elastic Beanstalk: migrate an on-prem app which uses SQS. Answer: use a docker container that includes the required code. Deploy that container into an Elastic Beanstalk env.
o	CFN: 1st deployment OK, but subsequent deployments have taken up to 15 minutes.
	cfn-hup.conf file has not been configured correctly. Lower the interval value to reduce deployment times.
	See https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-hup.html
	And https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/deploying.applications.html
CFN helper scripts:
•	Are written in Python
•	Install SW and start services on EC2 instances, created as part of a CFN stack.
•	Scripts send data back to CFN based on success/failure
•	Script locations: /opt/bin/aws

$ ls /opt/aws/bin/
cfn-elect-cmd-leader  cfn-hup   cfn-send-cmd-event   cfn-signal                eic_parse_authorized_keys
cfn-get-metadata      cfn-init  cfn-send-cmd-result  eic_curl_authorized_keys  eic_run_authorized_keys

typical related CFN template section:
"MyInstance": {
"Type": "AWS::EC2::Instance",
"Metadata": {
:
},
"Properties": {
:
"UserData" : { "Fn::Base64" : { "Fn::Join" : ["", [
"#!/bin/bash\n",
"yum update -y aws-cfn-bootstrap\n",
"/opt/aws/bin/cfn-init ",
" --stack ", { "Ref" : "AWS::StackName" },
" --resource MyInstance ",
" --region ", { "Ref" : "AWS::Region" }, "\n"
]]}}
}

cfn-hup.conf example from https://gist.github.com/libert-xyz/d1d7186ef3dc7ad23cbfe46eaf3fb27b :

            # The cfn-hup.conf file stores the name of the stack and the AWS credentials that the cfn-hup daemon targets.
            "/etc/cfn/cfn-hup.conf":
              content: !Sub |
                [main]
                stack=${AWS::StackId}
                region=${AWS::Region}
                # The interval used to check for changes to the resource metadata in minutes. Default is 15
                interval=2
              mode: "000400"
              owner: "root"
              group: "root"

Workshop on CFN helper scripts: https://catalog.workshops.aws/cfn101/en-US/basics/operations/helper-scripts 

Route 53
https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-values-weighted.html 
Value/Route traffic to

Choose IP address or another value depending on the record type. Enter a value that is appropriate for the value of Record type. For all types except CNAME, you can enter more than one value. Enter each value on a separate line.

You can route traffic to, or specify the following values:

    A record — IPv4 address
    AAAA — IPv6 address
    CAA — Certificate Authority Authorization
    CNAME record — Canonical name that points to the root, e.g. www.ex.com -> ex.com 
    MX — Mail exchange
    NAPTR — Name Authority Pointer
    PTR — Pointer
    SPF — Sender Policy Framework
    SRV — Service locator
    TXT — Text
Alias record: Route 53 responds with the value for that resource, e.g. API GW, VPC interf. Endpoint, CF distribution. Can be the top node of a DNS namespace (zone apex), e.g. example.com. This is not possible for a CNAME.


o	 
o	CFN Stacksets – get some experience building them. Especially cfn-init, cfn-hup
o	Elastic Beanstalk:
o	Applications, versions, in-place update, swap. .eb-extentions/config files!
o	EB can also use containers using ECS!
o	OpsWorks.
	Configuration management service – Offerings: Puppet, Chef or Stacks
	 
o	Containers:
	ECS, EKS, Fargate, EC2, ECR



20230628

CW monitoring (at least default) is available for:
https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/aws-services-cloudwatch-metrics.html - more than 130 services!


CW detailed monitoring is available for (see bottom of https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch-metrics-basic-detailed.html )

Service	Documentation
Amazon API Gateway	Dimensions for API Gateway metrics

Amazon CloudFront	Viewing additional CloudFront distribution metrics

Amazon EC2	Enable or turn off detailed monitoring for your instances

Elastic Beanstalk	Enhanced health reporting and monitoring

Amazon Kinesis Data Streams	Enhanced Shard-level Metrics

Amazon MSK	Amazon MSK Metrics for Monitoring with CloudWatch

Amazon S3	Amazon S3 request metrics in CloudWatch



Monitoring solution that sends a mail if certain errors (e.g. 404 show up in CW logs:

https://shahinahmed100.medium.com/how-to-query-aws-cloudwatch-logs-to-generate-alarm-using-lambda-9ebb8649c6ca

CW agent default config: /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.d/default

Commands to start and stop: https://bobcares.com/blog/status-of-cloudwatch-agent/

AMIs which have a pre-installed CW agent:

Typical reference flow to detect
 


Count 404 error codes using a CW metric filter:
https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/Counting404Responses.html 

CFN cfn-init and cloud-init (Ubuntu)

cloud-init
The cloud-init project tagline is that they are “The standard for customising cloud instances”. So it works with multiple cloud providers. It comes pre-installed in the Ubuntu Cloud Images and also on the official EC2 Ubuntu images. It has a larger documentation site: cloud-init Documentation.

The Cloud configs are written as YAML and allow you to use directives for common tasks like installing packages, configuring files, and running commands. Here are Cloud config examples Interestingly, there does not seem to be a service module.

The cloud-init scripts are stored in UserData in multipart format. They get expanded out to the /var/lib/cloud directory and then ran.

cfn-init
The cfn-init tool is an AWS officially supported way of customizing EC2 instances. It has simpler documentation. In fact there are really 2 main pages: AWS::CloudFormation::Init and cfn-init. It supports common configuration management tasks like creating packages, groups, users, sources, files, commands, services. These tasks are grouped together in “Configset”. Configsets are a part of a CloudFormation template, so they can be written as YAML or JSON.

The cfn-init script is pre-installed on the AmazonLinux2 distro. AWS packaged it up to make it easier to customize EC2 instances without having to take the additional step of installing a configuration management tool.

The cfn-init script typically pulls the Configset definition from the CloudFormation stack template itself. It can also be provided a JSON Configset definition directly from the file system.




20230421


DevOps Professional exam: version DOP-C02
https://aws.amazon.com/certification/certified-devops-engineer-professional/?ch=sec&sec=rmg&d=1
„Promo code reception“
https://home.pearsonvue.com/aws/freeretake?sc_channel=ha&sc_icampaign=aware_global_200_certification_ribbon_pv_examretake_B_tnc&sc_ichannel=ha&sc_icontent=awssm-409_aware&sc_iplace=ribbon&trk=21850408-1608-46ff-9af3-9c7cd33b7dbe~ha_awssm-409_aware 

Exam practice questions in Skillbuilder:
https://explore.skillbuilder.aws/pages/55/learner-dashboard?ctl231=l-_en~se-%22DOP-C02%22 
Comment from #aws-cert-prep: “There were couple questions on the exam which if not exactly, very similar to the questions from AWS's official practice exam. ”
Resulting Exam URL:
https://awscertificationpractice.benchprep.com/app/aws-certified-devops-engineer-professional-official-practice-exam-dop-c02?locale=en-us#exams/take_full/186453

Possible in Lambda: Managing provisioned concurrency with Application Auto Scaling  

Available in AWS Config: https://docs.aws.amazon.com/config/latest/developerguide/lambda-function-settings-check.html 

Possible in AWS Config: Multi-account, multi-Region data aggregation
Multi-account, multi-Region data aggregation is a capability on AWS Config that enables centralized auditing and governance. It gives you an enterprise-wide view of your AWS Config rule compliance status, and you can associate your AWS Organization to quickly add your accounts.

CodeBuild: You can use CodeBuild to run unit tests and build automation. CodeBuild can compile source code, run unit tests, and produce artifacts that are ready to deploy.

IAM: be cautious when allowing only access to recources e.g. in eu-west-1, because that means that global service access for e.g. IAM is blocked! Solution: “NotAction”: iam:*
 

Trusted Advisor – checks available in the Security area (e.g. “Exposed Access Keys”).
https://docs.aws.amazon.com/awssupport/latest/user/security-checks.html



CodeDeploy and Auto Scaling can be combined!
Prereq: install codeDeploy agent on EC2. An agent can pull in a new deployment from CodeDeploy.

From DynamoDB Streams, only two entities can consume: Kinesis Adapter and Lambda
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/streamsmain.html 

TutorialsDojo study guide:
Check the “Common Exam Scenarios” section to find scenarios and solutions for each domain!

Nice comparison of Eventbridge and SNS in the Eventbridge FAQs:
https://aws.amazon.com/eventbridge/faqs/

Again an even better comparison in AWS Blogs:
https://aws.amazon.com/blogs/compute/choosing-between-messaging-services-for-serverless-applications/ 

Amazon Simple Notification Service (Amazon SNS) can filter messages by attributes and not by message contents.


Brian Searles posted on March 27, 2023:

Brian Searles  25 days ago
Took and passed the DevOps Pro (DOP-C02) exam recently, and was recommended I share my observations about it here in case they're helpful for the larger audience. Comments in thread:
  2 2
10 replies
 
 
Brian Searles  25 days ago
My #1 protip for folks taking any cert is to have strong comfort with IAM mechanisms, and that wasn't any different here. Policies, Organizations, Control Tower, Roles, federating, and best practices were all on the exam multiple times and appeared more than any other concepts or services.
 
Brian Searles  25 days ago
There were numerous questions and solutions asking about EventBridge in some way, and a surprising number of questions about Config, sometimes pitting the two against each other, such as asking if the best way to find and do a thing would be to setup an EB rule, or to autoremediate via Config + SSM Automation or Lambda (or something). Having deeper appreciation of these mechanisms and their orchestration will help a lot, as the spirit of many questions turned into annoying variations of "which combination of the following integrations is right?" while looking for a single word or phrase that disqualifies an otherwise potentially viable path. I've always disliked the spirit of these questions as they are somewhat arbitrary in their ability to gauge depth, but it is what it is.
 
Brian Searles  25 days ago
I was surprised at the minimal amount of content relating to AWS container services, Lambda, and DynamoDB. To the extent Lambda was on the exam was in given solutions such as "create a lambda that does a thing, and invoke it via an EventBridge rule". Dynamo was in one or two of the responses but the exam featured RDS Aurora much more frequently around topics of Multi-AZ, cross region replicas, and Global Databases.
 
Brian Searles  25 days ago
Some prompts were really, really long. Like three paragraphs and six responses long. Timing around this exam is a little bananas at 75 questions and 3 hours, and I got fatigued. I needed every minute of the time just get through those types of questions and understand their intentions. I could have even used a few more minutes to go back and review all my marked questions. My advice isn't unlike others you've likely already seen: focus and read for comprehension the first time though, think about what answers "should" work, don't overcomplicate your first draft at answers, and move on. It's better to cap your efforts at a couple minutes initially rather than spending a full 5 or 6, because on review you'll have the added benefit of all the other questions exposing you to additional potential ideas and answers, and a long and wordy question is worth the same credit a short and easy one is.
 
Brian Searles  25 days ago
As you'd expect, emphasis of Code* services is important. If I had to do this exam over again, I'd have gone back and done every tutorial and workshop I could have gotten my hands on for these services, particularly for CodeBuild and CodePipeline, as I was weak in this area before and it went a bit deeper than I thought it would. As an example of depth, there were questions on creating Code* resiliency across regions, or in optimizing CodeBuild and CodePipeline routines. I knew these services at a high level but not as well as I ought to have.
 
Terrell Jackson   25 days ago
Thank you for sharing because I am studying for it now.
 
Brian Searles  25 days ago
I got the cert, but didn't feel pretty about it and I know I could have done better in both prep and execution. For the record, I completed the sample questions and practice question set from Skillbuilder, and my scores were spot-on from the averages I got there. I did not use any third party tools to help, but I wish I had.Hopefully this talk helps someone else out there for their own future efforts. And, naturally, my experiences here are just my own, so even though I didn't happen to see any really detailed Lambda or Dynamo questions, that doesn't mean you might not in your attempts.
    6 2
 
Terrell Jackson   25 days ago
Congratulations
 1
 
Kerith Salmon  25 days ago
@bsearle did you use any third-party prep material?
 
Brian Searles  25 days ago
I did not, but on reflection wish I had.




Sathya Kannappan  1 month ago
@bahuga,
I recently archived AWS Certified DevOps Engineer Professional 2023 (DOP-C02) certification. I would recommend to go through
1. Stephane Maarek's AWS Certified DevOps Engineer Professional 2023 - Hands On!
2. Exam Readiness - https://explore.skillbuilder.aws/learn/course/74/exam-readiness-aws-certified-devops-engineer-professional
3. An updated practice question from Skillbuilder: https://explore.skillbuilder.aws/learn/course/14673/aws-certified-devops-engineer-professional-official-practice-question-set-dop-c02-english
4. Practice Exam - https://explore.skillbuilder.aws/learn/course/internal/view/elearning/14959/aws-certified[…]ofessional-official-practice-exam-dop-c02-english-amazon
5. FAQs for ASG, ALB, SQS, Lambda, CodeDeploy, Cloudwatch Metric, Eventbridge, EKS, ElasticBeanstack, SSM, CodePipeline
explore.skillbuilder.aws
Self-paced digital training on AWS - AWS Skill Builder
Your learning center to build in-demand cloud skills.
explore.skillbuilder.aws
Self-paced digital training on AWS - AWS Skill Builder
Your learning center to build in-demand cloud skills.
explore.skillbuilder.aws
Self-paced digital training on AWS - AWS Skill Builder
Your learning center to build in-demand cloud skills.


https://explore.skillbuilder.aws/learn/course/14673/aws-certified-devops-engineer-professional-official-practice-question-set-dop-c02-english

•	
Refresh of https://jayendrapatil.com/aws-certified-devops-engineer-professional-exam-learning-path/




end of document
