To enable **passwordless SSH authentication** between two machines (e.g., your local WSL/Ansible control node and an AWS EC2 instance), follow these steps:

---

## ✅ Step-by-Step: Passwordless SSH Authentication

### 🔹 1. **Generate SSH Key Pair (if not already done)**

On your **local machine (WSL or Ansible control node)**:

```bash
ssh-keygen
```

* Press Enter at each prompt to accept the defaults.
* This creates:

  * Private key: `~/.ssh/id_rsa`
  * Public key: `~/.ssh/id_rsa.pub`

---

### 🔹 2. **Copy Public Key to Remote EC2 Instance**

You need to add your public key to the remote EC2's `authorized_keys` file.

Use one of these two methods:

#### ✅ Option A: Using `ssh-copy-id` (recommended if available)

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub -o "IdentitiesOnly=yes" -o "StrictHostKeyChecking=no" ubuntu@<EC2_PUBLIC_IP>
```

Replace `<EC2_PUBLIC_IP>` with your instance’s public IP.

If this works, you're done. Skip to **Step 3**.

---

#### ✅ Option B: Manually (if `ssh-copy-id` is not available)

1. First, ensure the `.ssh` directory exists on EC2:

```bash
ssh -i awskey.pem ubuntu@<EC2_PUBLIC_IP> "mkdir -p ~/.ssh && chmod 700 ~/.ssh"
```

2. Then, copy your public key to the EC2 instance:

```bash
cat ~/.ssh/id_rsa.pub | ssh -i awskey.pem ubuntu@<EC2_PUBLIC_IP> "cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

> ⚠️ Only do this once, or it will duplicate entries.

---

### 🔹 3. **Test Passwordless SSH Login**

Try logging into your EC2 instance:

```bash
ssh ubuntu@<EC2_PUBLIC_IP>
```

* It should log in **without asking for a password** or `.pem` file.

---

## 🛠️ Troubleshooting

* **File permissions must be correct:**

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

* **Make sure your SSH agent is running (if using it)**:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

---

## 🧠 Summary

| Step             | Command                                 |
| ---------------- | --------------------------------------- |
| Generate SSH Key | `ssh-keygen`                            |
| Copy Key         | `ssh-copy-id ubuntu@<EC2_IP>` or manual |
| Test             | `ssh ubuntu@<EC2_IP>`                   |

---

Would you like to automate this entire process with a script or Ansible playbook?
