# Contributing to AWS Learning Repository

First off, thank you for considering contributing to this AWS learning repository! It's people like you that make this resource valuable for learners worldwide.

## How Can I Contribute?

### Reporting Issues

If you find any errors, outdated information, or have suggestions:

1. Check if the issue already exists in [GitHub Issues](https://github.com/krishnakrish7/aws-learning-/issues)
2. If not, create a new issue with:
   - Clear, descriptive title
   - Detailed description of the problem or suggestion
   - Steps to reproduce (if applicable)
   - Expected vs actual behavior
   - AWS region and service version (if relevant)

### Suggesting Enhancements

Enhancement suggestions are tracked as GitHub issues:

1. Use a clear, descriptive title
2. Provide a detailed description of the enhancement
3. Explain why this would be useful to learners
4. Include examples if possible

### Pull Requests

1. **Fork the repository**
   ```bash
   git clone https://github.com/krishnakrish7/aws-learning-.git
   cd aws-learning-
   ```

2. **Create a branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **Make your changes**
   - Follow the existing structure and style
   - Test any code examples
   - Update documentation as needed

4. **Commit your changes**
   ```bash
   git add .
   git commit -m "Add: description of your changes"
   ```

5. **Push to your fork**
   ```bash
   git push origin feature/your-feature-name
   ```

6. **Create a Pull Request**
   - Go to the original repository
   - Click "New Pull Request"
   - Select your branch
   - Fill in the PR template

## Content Guidelines

### Documentation Style

- Use clear, concise language
- Explain concepts before showing code
- Include practical examples
- Add comments to code snippets
- Link to official AWS documentation

### Code Examples

- Test all code before submitting
- Use current AWS best practices
- Include error handling where appropriate
- Add comments for complex logic
- Specify AWS region when relevant
- Use placeholder values (e.g., `YOUR_BUCKET_NAME`, `123456789012`)

### File Organization

```
service-name/
├── README.md           # Main documentation
├── examples/           # Code examples
│   ├── basic/
│   └── advanced/
├── tutorials/          # Step-by-step guides
└── scripts/            # Utility scripts
```

### Markdown Standards

- Use headers hierarchically (H1 → H2 → H3)
- Include code language specifiers in fenced code blocks
- Use relative links for internal references
- Add alt text for images
- Use tables for structured data
- Include a table of contents for long documents

### AWS-Specific Guidelines

1. **Regions**: Specify when examples are region-specific
2. **Costs**: Warn about potential costs
3. **Permissions**: Document required IAM permissions
4. **Security**: Follow AWS security best practices
5. **Free Tier**: Indicate Free Tier eligibility
6. **Cleanup**: Always include cleanup instructions

## Types of Contributions

### 1. New Service Documentation

When adding a new AWS service:

- Create a folder under appropriate category
- Include comprehensive README.md
- Add basic and advanced examples
- Link to official AWS documentation
- Update main README.md

### 2. Tutorials

- Step-by-step instructions
- Clear prerequisites
- Expected outcomes
- Cleanup procedures
- Troubleshooting section

### 3. Code Examples

- Well-commented code
- Multiple language implementations (when possible)
- Error handling
- Best practices demonstrated

### 4. Best Practices

- Based on AWS Well-Architected Framework
- Include rationale
- Provide examples
- Link to official guidelines

### 5. Troubleshooting Guides

- Common errors and solutions
- Debug techniques
- Links to resources

## Code Style

### Bash Scripts

```bash
#!/bin/bash

# Description of what script does
# Author: Your Name
# Date: YYYY-MM-DD

set -e  # Exit on error

# Variables
REGION="us-east-1"
BUCKET_NAME="example-bucket"

# Functions
function create_bucket() {
    aws s3 mb "s3://${BUCKET_NAME}" --region "${REGION}"
}

# Main execution
create_bucket
```

### Python

```python
"""
Module description
"""

import boto3
from botocore.exceptions import ClientError


def example_function(bucket_name: str) -> dict:
    """
    Function description.
    
    Args:
        bucket_name: Name of the S3 bucket
        
    Returns:
        Dictionary with operation results
    """
    s3_client = boto3.client('s3')
    
    try:
        response = s3_client.list_objects_v2(Bucket=bucket_name)
        return response
    except ClientError as e:
        print(f"Error: {e}")
        raise
```

### CloudFormation/JSON

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Template description'

Parameters:
  BucketName:
    Type: String
    Description: Name for the S3 bucket

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref BucketName
```

## Testing

Before submitting:

1. **Test code examples**
   - Run in your AWS account
   - Verify output matches documentation
   - Test in Free Tier when possible

2. **Check links**
   - Verify all links work
   - Use HTTPS where available
   - Prefer official AWS documentation

3. **Verify formatting**
   - Markdown renders correctly
   - Code blocks have proper syntax highlighting
   - Tables display correctly

4. **Security check**
   - No hardcoded credentials
   - No sensitive information
   - Follow security best practices

## Commit Message Guidelines

Use conventional commits format:

```
type(scope): subject

body

footer
```

**Types:**
- `Add:` New content or features
- `Update:` Updates to existing content
- `Fix:` Bug fixes or corrections
- `Docs:` Documentation only changes
- `Style:` Formatting changes
- `Refactor:` Code restructuring
- `Remove:` Removing content

**Examples:**
```
Add: EC2 auto-scaling tutorial

Update: S3 lifecycle policy examples

Fix: Incorrect CloudFormation syntax in Lambda example

Docs: Improve getting started guide clarity
```

## Review Process

1. Maintainer reviews PR
2. Feedback provided if changes needed
3. Once approved, PR is merged
4. Contributor is credited

## Recognition

Contributors will be recognized in:
- GitHub contributors list
- Release notes (for significant contributions)
- Special thanks section (if we add one)

## Questions?

Feel free to:
- Open an issue for questions
- Reach out to maintainers
- Join discussions in issues

## Code of Conduct

### Our Pledge

We are committed to providing a welcoming and inspiring community for all.

### Our Standards

- Be respectful and inclusive
- Accept constructive criticism gracefully
- Focus on what's best for the community
- Show empathy towards others

### Unacceptable Behavior

- Harassment or discrimination
- Trolling or insulting comments
- Public or private harassment
- Publishing others' private information
- Other unethical or unprofessional conduct

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

## Thank You!

Your contributions help others learn AWS. Every contribution, no matter how small, is appreciated!

---

Happy Contributing! 🚀
