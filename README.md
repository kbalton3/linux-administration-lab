# Linux System Administration Lab

## Project Overview

This project demonstrates hands-on Linux system administration using an Ubuntu virtual machine running in UTM on an Apple Silicon Mac. The lab was designed to simulate common responsibilities performed by Linux administrators, system administrators, and IT support professionals.

Throughout the project, I configured user accounts and groups, implemented file permissions and access controls, deployed an Apache web server, configured UFW firewall rules, enabled SSH remote administration, monitored system resources, analyzed system and authentication logs, automated backups with cron, managed software updates, audited network services, and examined disk and filesystem utilization.

The lab also required troubleshooting real configuration issues, including resolving system time synchronization problems that prevented Ubuntu repositories from updating correctly.

---

## Lab Environment

- **Operating System:** Ubuntu Linux
- **Virtualization:** UTM
- **Host System:** Apple Silicon Mac
- **Shell:** Bash
- **Web Server:** Apache2
- **Remote Administration:** OpenSSH
- **Firewall:** UFW
- **Package Manager:** APT
- **Logging:** systemd journal / journalctl
- **Automation:** cron

---

## Objectives

The objectives of this lab were to:

- Develop practical Linux command-line administration skills
- Manage Linux users, groups, permissions, and protected resources
- Configure and manage system services
- Deploy and verify an Apache web server
- Configure host-based firewall rules
- Enable and test secure remote administration
- Monitor system resources and network services
- Analyze system and authentication logs
- Automate administrative tasks
- Perform package and update management
- Troubleshoot Linux system issues
- Validate security controls through hands-on testing

---

## 1. System and Network Verification

I began by verifying the Linux environment and network configuration. This established a baseline before making administrative changes to the system.

![System and Network Verification](screenshots/01-System-Network-Verification.png)

---

## 2. System Update and Package Management

APT was used to refresh Ubuntu software repositories and identify available package updates.

During this portion of the lab, an incorrect system clock caused repository metadata to report that release files were not yet valid. I investigated the issue using `date` and `timedatectl`, identified that the system clock was not synchronized, restored NTP synchronization, and successfully reran the package update.

![System Update Verification](screenshots/02-System-Update-Verification.png)

![Package Update Management](screenshots/16-Package-Update-Management.png)

---

## 3. User and Group Administration

Multiple Linux accounts were created to practice identity and access administration. Group membership was then used to control access to shared system resources.

The `labadmins` group was configured to provide authorized access to a protected directory. The `labuser` account was added to the group while `guestuser` remained unauthorized.

![User Account Verification](screenshots/03-User-Account-Verification.png)

![Group Membership](screenshots/04-Group-Membership.png)

---

## 4. File Permissions and Access Control

A protected directory named `/shared-lab` was configured with group-based permissions.

The directory was owned by:

```text
root:labadmins
```

Permissions were configured as:

```text
drwxrwx---
```

This corresponds to permission mode `770`, allowing access to the owner and authorized group while preventing access by other users.

Access testing confirmed that `labuser` could access the protected directory while `guestuser` received a `Permission denied` response.

![Directory Permissions](screenshots/05-Directory-Permissions.png)

![Linux Group Permissions Access Test](screenshots/06-Linux-Group-Permissions-Access-Test.png)

![Access Control Permissions Audit](screenshots/20-Access-Control-Permissions-Audit.png)

---

## 5. Apache Web Server Administration

Apache2 was installed and managed as a Linux system service. I verified that the service was running through `systemctl` and deployed a custom webpage to confirm successful web-server operation.

![Apache Service Running](screenshots/07-Apache-Service-Running.png)

![Apache Web Server Test](screenshots/08-Apache-Web-Server-Test.png)

---

## 6. Firewall Configuration

UFW was configured as the host-based firewall.

The firewall was configured to deny unsolicited incoming traffic while allowing required services. HTTP traffic was permitted on TCP port 80, and OpenSSH was permitted for remote administration.

![UFW Firewall Configuration](screenshots/09-UFW-Firewall-Configuration.png)

![Firewall HTTP and SSH Rules](screenshots/11-Firewall-HTTP-SSH-Rules.png)

---

## 7. SSH and Remote Administration

OpenSSH Server was installed, enabled, and verified as an active system service.

I then connected from the host Mac to the Ubuntu VM over SSH, demonstrating remote Linux administration.

The remote session verified the Linux username, hostname, and network address.

![SSH Service Running](screenshots/10-SSH-Service-Running.png)

![Remote SSH Administration](screenshots/12-Remote-SSH-Administration.png)

---

## 8. System Resource Monitoring

Linux command-line utilities were used to examine system health and resource utilization, including memory, storage, uptime, load, and running processes.

Commands used included:

```bash
free -h
df -h
uptime
ps aux --sort=-%mem | head
```

Apache and SSH processes were also verified from the process table.

![System Resource Monitoring](screenshots/13-System-Resource-Monitoring.png)

---

## 9. System and Authentication Log Analysis

System logs were reviewed using `journalctl` and authentication logs.

This provided visibility into:

- SSH service activity
- Apache service events
- Successful remote authentication
- User sessions
- Administrative activity

![System Log Analysis](screenshots/14-System-Log-Analysis.png)

---

## 10. Automated Backup Administration

A scheduled backup process was configured using cron.

The backup archives the protected `/shared-lab` directory:

```bash
tar -czf /home/kenise/shared-lab-backup.tar.gz /shared-lab
```

Because the directory is protected by group permissions, the scheduled task was configured in the root crontab so it could access the protected files.

The automated task was configured to run daily at 2:00 AM:

```text
0 2 * * * tar -czf /home/kenise/shared-lab-backup.tar.gz /shared-lab 2>/home/kenise/backup-error.log
```

The resulting archive was manually verified to ensure that `labuser-test.txt` was successfully included.

![Automated Backup Verification](screenshots/15-Automated-Backup-Verification.png)

---

## 11. Network Service Auditing

Active listening services were audited using:

```bash
sudo ss -tulpn
```

The audit confirmed that:

- SSH was listening on TCP port **22**
- Apache was listening on TCP port **80**

The listening services were compared with UFW rules to confirm that firewall access aligned with the services intentionally exposed by the system.

![Network Service Audit](screenshots/17-Network-Service-Audit.png)

---

## 12. Login and SSH Security Auditing

Login history and SSH authentication events were reviewed to validate remote-access activity.

The `last` command was used to review login sessions, while `journalctl` was used to inspect SSH authentication events.

The logs confirmed a successful remote SSH authentication and session creation.

![Login Security Audit](screenshots/18-Login-Security-Audit.png)

---

## 13. Disk and Filesystem Administration

Disk devices, partitions, mount points, and filesystem utilization were examined using `lsblk` and `df`.

The lab VM contained a 30 GB virtual disk with separate root and EFI partitions. Disk utilization was reviewed to determine available storage capacity.

![Disk and Filesystem Administration](screenshots/19-Disk-Filesystem-Administration.png)

---

## Troubleshooting and Lessons Learned

One of the most valuable parts of this project was troubleshooting issues rather than simply executing commands.

### System Time Synchronization

APT initially failed to apply repository updates because Ubuntu reported that several repository release files were "not valid yet."

I investigated the system clock using:

```bash
date
timedatectl
```

The output showed:

```text
System clock synchronized: no
NTP service: active
```

NTP synchronization was reset and re-enabled. Verification afterward showed:

```text
System clock synchronized: yes
NTP service: active
```

APT subsequently contacted all configured repositories successfully.

### Backup Permissions

An initial user-level backup attempt could not properly archive `/shared-lab` because the account running the task did not have permission to access the protected directory.

After reviewing the directory permissions, I moved the scheduled backup to the root crontab and verified the archive contents.

These troubleshooting exercises reinforced the importance of understanding permissions, services, system time, logs, and command output rather than relying only on whether a command executes.

---

## Skills Demonstrated

- Linux system administration
- Ubuntu administration
- Bash command line
- User and group management
- Linux file permissions
- Group-based access control
- Apache web server administration
- systemd service management
- SSH configuration and remote administration
- UFW firewall configuration
- Network service auditing
- APT package management
- System resource monitoring
- Process monitoring
- System and authentication log analysis
- cron task scheduling
- Backup administration
- Disk and filesystem administration
- Troubleshooting
- Basic Linux security hardening

---

## Key Takeaways

This project strengthened my understanding of Linux administration by requiring me to configure, test, troubleshoot, and validate services rather than only study Linux concepts theoretically.

The lab demonstrated how user administration, permissions, networking, services, logging, automation, patch management, and security controls work together within a Linux environment. It also provided practical experience diagnosing configuration problems and verifying that implemented controls behaved as intended.

---

## Screenshot Evidence

Additional evidence for each stage of the lab is available in the [`screenshots`](screenshots/) directory.
