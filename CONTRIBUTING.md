# Contributing to Exim SMTP Relay Ansible Playbook

Thank you for your interest in contributing! This document provides guidelines for contributing to this project.

## How to Contribute

### Reporting Bugs

If you find a bug, please create an issue with:

1. **Clear title**: Describe the issue briefly
2. **Description**: Detailed explanation of the problem
3. **Steps to reproduce**: How to recreate the issue
4. **Expected behavior**: What should happen
5. **Actual behavior**: What actually happens
6. **Environment**:
   - Ansible version
   - Ubuntu version
   - Exim version
   - Any relevant configuration

### Suggesting Enhancements

For feature requests:

1. Check if the feature already exists
2. Check if someone already suggested it
3. Create an issue with:
   - Clear description of the feature
   - Use cases
   - Potential implementation approach

### Pull Requests

1. **Fork the repository**
2. **Create a feature branch**: `git checkout -b feature/your-feature-name`
3. **Make your changes**:
   - Follow the coding standards
   - Add comments for complex logic
   - Update documentation
4. **Test your changes**:
   - Test on Ubuntu 24.04 LTS
   - Use `ansible-lint` to check playbook
   - Verify idempotency
5. **Commit your changes**:
   - Use clear, descriptive commit messages
   - Reference issues if applicable
6. **Push to your fork**: `git push origin feature/your-feature-name`
7. **Create a Pull Request**:
   - Provide clear description
   - Reference related issues
   - Include testing results

## Coding Standards

### Ansible Best Practices

- Use fully qualified collection names (e.g., `ansible.builtin.apt`)
- Include descriptive task names
- Use tags appropriately
- Keep tasks idempotent
- Use handlers for service restarts
- Include appropriate error handling

### YAML Style

```yaml
# Good
- name: Install packages
  ansible.builtin.apt:
    name:
      - package1
      - package2
    state: present

# Avoid
- name: Install packages
  apt: name=package1,package2 state=present
```

### Documentation

- Update README.md for major changes
- Add comments in templates
- Update USAGE.md for new features
- Include examples for new functionality

### Testing

Before submitting:

```bash
# Lint the playbook
ansible-lint playbook.yml

# Test syntax
ansible-playbook playbook.yml --syntax-check

# Dry run
ansible-playbook -i inventory.ini playbook.yml --check

# Full test on clean Ubuntu 24.04
ansible-playbook -i inventory.ini playbook.yml
```

## Development Setup

```bash
# Clone your fork
git clone https://github.com/YOUR_USERNAME/ansible-exim-smtp-relay.git
cd ansible-exim-smtp-relay

# Add upstream remote
git remote add upstream https://github.com/d13g0s0uz4/ansible-exim-smtp-relay.git

# Install development dependencies
pip install ansible-lint yamllint

# Create test environment (optional)
vagrant init ubuntu/noble64
```

## Areas for Contribution

We welcome contributions in these areas:

### High Priority

- [ ] Add support for DKIM signing
- [ ] Implement SPF validation
- [ ] Add ClamAV virus scanning
- [ ] Integrate SpamAssassin
- [ ] Add monitoring and alerting
- [ ] Create Molecule tests

### Medium Priority

- [ ] Support for multiple Linux distributions
- [ ] LDAP authentication integration
- [ ] Advanced logging configuration
- [ ] Backup and restore functionality
- [ ] Performance tuning options

### Low Priority

- [ ] IPv6 support enhancements
- [ ] Custom ACL templates
- [ ] Integration with external monitoring systems
- [ ] Ansible Galaxy role conversion

## Code Review Process

1. Maintainers will review PRs within 7 days
2. Feedback will be provided via PR comments
3. Changes may be requested
4. Once approved, changes will be merged
5. Contributors will be acknowledged

## Communication

- **Issues**: For bugs and feature requests
- **Pull Requests**: For code contributions
- **Discussions**: For questions and ideas

## Recognition

Contributors will be:

- Listed in the repository contributors
- Mentioned in release notes
- Credited in commit messages

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

Thank you for contributing! 🚀