# Creating-A-DynamoDB-Table-Via-CLI

Scenario: Let’s Go! Media Inc. stores media content data in AWS DynamoDB. They created a table called “MediaCatalog” and added information about 10 movies. An IAM role with read-only permissions ensures security. Only authorized users can use the AWS CLI to scan the table.

Project Overview: This project has 3 tiers of difficulty. Foundational, Advanced and Complex. I will attempting all 3 tiers and documenting the steps to accomplish them.

The tasks for Tier 1 are as follows:

— Create a DynamoDB table for latest Movie releases
— Add 10 latest movie releases to the table, including the title, genre, release date, and rating.
— Create a t.2micro EC2 instance
— Using an IAM role and the principle of least privilege, grant the EC2 instance read access to DynamoDB.
— Use the AWS CLI in the EC2 instance to scan the DynamoDB table
— Use the AWS CLI in the EC2 instance to validate you cannot write an item to the DynamoDB table

Let’s get started!!!

# Tier 1: Foundational

The first step is to create a DynamoDB table and name it MediaCatalog. Let’s head over to the DynamoDB console and click on Create table. Provide the name and Partition key. Everything else I will be leaving as default.

![Snipe 1](https://github.com/Mirahkeyz/Creating-A-DynamoDB-Table-Via-CLI/assets/134533695/72b0d1a2-0cf7-4d70-afd5-ee7a3fb85ea8)

After Creating the Table We can now start to create items in the table. Click on Explore items then Create item. Since the table has to do with the last 10 movies released, I will be using Asteroid City, Barbie, Guardians 3, Indiana Jones, Mission Impossible, Oppenheimer, Spider-verse, The flash, The Outlaws and Transformers.

![Snipe 2](https://github.com/Mirahkeyz/Creating-A-DynamoDB-Table-Via-CLI/assets/134533695/e9d4f7ad-cc52-4abe-92a5-1911a6e0c6bb)

![Snipe 3](https://github.com/Mirahkeyz/Creating-A-DynamoDB-Table-Via-CLI/assets/134533695/9526b7c8-7ecc-48ae-a34a-bd2a26e3bf3d)

The database has now been created. Next thing on the list is to launch an EC2 instance. The instance will be a t2.micro Amazon Linux EC2.

Now head over to IAM and click on Roles, Create role. Select AWS service, EC2 then Next. Type in DynamoDB in the search bar and hit enter.

We are going to choose AmazonDynamoDBReadOnlyAccess and provide a name in the next screen.

![Snipe 4](https://github.com/Mirahkeyz/Creating-A-DynamoDB-Table-Via-CLI/assets/134533695/7ed7daf6-561b-44fb-9840-2cb9950c57bd)

![Snipe 5](https://github.com/Mirahkeyz/Creating-A-DynamoDB-Table-Via-CLI/assets/134533695/fb19802f-9806-4587-a208-0f12fd3040c0)

![Snipe 6](https://github.com/Mirahkeyz/Creating-A-DynamoDB-Table-Via-CLI/assets/134533695/33ed8137-1e55-41ef-ae9f-c25383ce1cc0)

Now scroll down and click Create role. Head back over to the EC2 console and right click the EC2 instance we created and select Security, Modify IAM role.

Choose the read only role we just created and click on Update IAM role.

![Snipe 7](https://github.com/Mirahkeyz/Creating-A-DynamoDB-Table-Via-CLI/assets/134533695/365941d6-012f-4310-b48b-7a198190da20)

Let’s log into the instance to verify that we can only read and not write to the DynamoDB table.

aws dynamodb query --table-name MediaCatalog --key-condition-expression "#MT = :title" --expression-attribute-names '{"#MT": "Movie Title"}' --expression-attribute-values '{":title": {"S": "Transformers: Rise of the Beasts"}}'

![Snipe 8](https://github.com/Mirahkeyz/Creating-A-DynamoDB-Table-Via-CLI/assets/134533695/15bfd666-3fd8-489c-9427-fb2a09cac642)

We now know that we can read the table. Next, we have to verify that we cannot write to the table. To test this out we can run the following command:

aws dynamodb put-item --table-name MediaCatalog --item '{"MovieTitle": {"S": "Gremlins"}, "Genre": {"S": "Horror"},"Rating": {"S": "PG"},"Release Date": {"S": "1984-06-08"}}'

![Snipe 9](https://github.com/Mirahkeyz/Creating-A-DynamoDB-Table-Via-CLI/assets/134533695/803935d0-e607-4b3c-bfaf-ca10165c23c1)

As we can see, we are only able to read DynamoDB tables due to the role that is attached.

We now have completed Tier 1. Let’s head over to Tier 2.

# Tier 2: Advanced

The task for Tier 2 is to create everything from Tier 1 via the CLI.

First thing we need to create the profile

aws configure --profile leveluptech

Then we need to create the table. We can run the following command: (replace the profile name at the end of the command with your own)
 
 aws dynamodb create-table --table-name MediaCatalog --attribute-definitions AttributeName=MovieTitle,AttributeType=S --key-schema AttributeName=MovieTitle,KeyType=HASH --billing-mode PAY_PER_REQUEST --profile leveluptech      

aws dynamodb create-table --table-name MediaCatalog --attribute-definitions AttributeName=MovieTitle,AttributeType=S --key-schema AttributeName=MovieTitle,KeyType=HASH --billing-mode PAY_PER_REQUEST --profile leveluptech

![Snipe 10](https://github.com/Mirahkeyz/Creating-A-DynamoDB-Table-Via-CLI/assets/134533695/d91ee054-8096-4b71-a223-4edd7dd6ba37)

Now we need to import some data into the table. I created a json file with the data. We just need to import it via the CLI. We can import it by running the following command:

aws dynamodb batch-write-item --request-items file://mediacatalog.json --profile leveluptech

Now to verify the table contents:

aws dynamodb scan --table-name MediaCatalog --profile leveluptech

![Snipe 11](https://github.com/Mirahkeyz/Creating-A-DynamoDB-Table-Via-CLI/assets/134533695/e0abd57d-cd79-4207-93b1-4ae8a5a82229)

The table has been created and now we need to create the EC2 instance via the CLI. Please run the command:

aws ec2 run-instances --image-id ami-0f34c5ae932e6f0e4 --instance-type t2.micro --security-group-ids sg-023a1bae0002da860 --subnet-id subnet-0571a15f8b1e7df22 --key-name AmazonLinux --profile leveluptech

![Snipe 12](https://github.com/Mirahkeyz/Creating-A-DynamoDB-Table-Via-CLI/assets/134533695/9b02784a-9b11-4455-af03-7621fcb36e4a)

It’s time to attach the AmazonDynamoDBReadOnlyAccess IAM role. Run this command to get the instance ID:

aws ec2 describe-instances --profile leveluptech

![Snipe 13](https://github.com/Mirahkeyz/Creating-A-DynamoDB-Table-Via-CLI/assets/134533695/18770929-54db-484d-9f94-6972ef4c4592)

Copy the instance ID and use the following command to attach the role:

aws ec2 associate-iam-instance-profile --instance-id i-07449b1b2a5c228f1 --iam-instance-profile Name="DynamoDBReadOnly" --profile leveluptech

If it ran successfully, you should receive the following message:

![Snipe 14](https://github.com/Mirahkeyz/Creating-A-DynamoDB-Table-Via-CLI/assets/134533695/22ee4e38-ce6f-46f9-86b5-09aed704b53f)

Let’s log into the EC2 instance and verify that we are unable to write to the table.

We can view the table…..

![Snipe 15](https://github.com/Mirahkeyz/Creating-A-DynamoDB-Table-Via-CLI/assets/134533695/25830f1d-14b7-4cbf-9638-4a619f72b728)

Verified that we cannot write but only read the table:

![Snipe 16](https://github.com/Mirahkeyz/Creating-A-DynamoDB-Table-Via-CLI/assets/134533695/576ba544-34e2-4dea-a5fc-597c5fbab5b2)


































































































