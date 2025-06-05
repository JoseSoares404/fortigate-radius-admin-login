# FortiGate Admin Login via FreeRADIUS (AlmaLinux 9 Lab) - By Jose Soares

## Overview
This lab demonstrates how to configure centralized RADIUS authentication for FortiGate admin login using a FreeRADIUS server running on AlmaLinux 9. The result is a more secure, role-based access model aligned with enterprise standards.
![image](https://github.com/user-attachments/assets/bc93d687-62ff-4c09-99fc-ebb99250c95c)

## 🎯 Purpose of This Lab
This lab is part of my **Fortinet Certified Professional (FCP)** certification prep. Remote Authentication is a key topic in the curriculum — and what better way to truly understand it than to build it from scratch. Rather than just memorize concepts, I wanted to gain hands-on experience by implementing FreeRADIUS, configuring FortiGate as a RADIUS client, and enabling admin login using remote credentials. This project deepened my understanding of authentication protocols, firewall admin policies, and secure network access design.

![image](https://github.com/user-attachments/assets/0cadacde-1fe0-4462-bcdb-5c28d7d14e58)



---

## Lab Stack
- **Firewall:** FortiGate 40F (v7.2.11)
- **RADIUS Server:** FreeRADIUS 3 on AlmaLinux 9
- **Topology:**

![image](https://github.com/user-attachments/assets/3f2b6273-553f-4bbc-9db6-4ce600de5552)


---

## 🛠️ Configuration Steps

### 1. Install FreeRADIUS on AlmaLinux 9
```bash
sudo dnf install freeradius freeradius-utils -y
```

### 2. Allow FortiGate as a RADIUS client
Edit `/etc/raddb/clients.conf`:
```conf
client fortigate40f {
    ipaddr = 192.168.10.99
    secret = cisco
}
```

### 3. Create RADIUS user
Edit `/etc/raddb/users`:
```conf
jose.soares Cleartext-Password := "cisco"
    Fortinet-Group-Name := "Radius-Admins"
```

### 4. Open firewall ports (if firewalld is active)
```bash
sudo firewall-cmd --add-port=1812/udp --permanent
sudo firewall-cmd --add-port=1813/udp --permanent
sudo firewall-cmd --reload
```

### 5. Run FreeRADIUS in debug mode
```bash
sudo systemctl stop radiusd
sudo radiusd -X
```

---

## 🔧 FortiGate Configuration

### 1. Create RADIUS Server
- Go to **User & Authentication > RADIUS Servers**
- Add new:
  - IP/Name: `192.168.10.44`
  - Secret: `cisco`
  - NAS IP: `192.168.10.99`
  - Test connectivity (should show ✅)

### 2. Create User Group
- Go to **User & Authentication > User Groups**
- Create `Radius-Admins`
  - Type: Firewall
  - Remote Server: `FreeRADIUS-LAB`

### 3. Create Admin Account
- Go to **System > Administrators**
- Create New:
  - Type: *Match a user on a remote server group*
  - Username: `jose.soares`
  - Remote Group: `Radius-Admins`
  - Profile: `super_admin` (or `readonly`)

---

## ✅ Result
FortiGate now authenticates `jose.soares` as a remote admin using FreeRADIUS, mapped to a Fortinet admin group via `Fortinet-Group-Name` attribute.

---

## 📸 Screenshots
- `radiusd -X` showing successful login
![image](https://github.com/user-attachments/assets/14679a79-76c4-4278-b674-fc61e1a22285)
![image](https://github.com/user-attachments/assets/53613918-b33a-4a2e-ac9c-821fee4b3afb)





- FortiGate "Test User Credentials" success
![image](https://github.com/user-attachments/assets/87b0b465-4973-42c5-b463-8d43e4313ab2)
![image](https://github.com/user-attachments/assets/57810c62-4a59-40ed-97d4-7b77a4a23b64)
![image](https://github.com/user-attachments/assets/f3a02022-74e5-4d98-a0a6-f25c2c8fb428)


- RADIUS user config and group mapping
![image](https://github.com/user-attachments/assets/ab6deb3b-bb29-439a-9589-9f9cd6520811)


---

## 💡 Lessons Learned
- FreeRADIUS debugging with `radiusd -X`
- Proper use of `Fortinet-Group-Name` for role mapping
- FortiGate RADIUS integration for admin logins

---

## 🔗 Ideas for further labs using this same setup
- Role-based access control (RBAC)
- 802.1X / WiFi authentication via RADIUS
- Integrating FreeRADIUS with LDAP or Active Directory (this would be fun!)

---

## 📁 Repo Files
- `users` – my RADIUS users file
- `clients.conf` – FortiGate RADIUS client config
- `screenshots/` – screenshots of this lab steps and results
