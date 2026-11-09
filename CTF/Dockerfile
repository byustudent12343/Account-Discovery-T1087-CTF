# =============================================================================
#  Acme Corp "acme-web01" - practice lab for MITRE ATT&CK T1087.001
#  (Account Discovery: Local Account)
#
#  This image is a deliberately simple, harmless training target. It contains
#  only fake user accounts for students to enumerate. No real services run,
#  and no exploits are included.
# =============================================================================
FROM ubuntu:24.04

LABEL description="T1087.001 Local Account Discovery practice lab (Acme Corp acme-web01)"

# -----------------------------------------------------------------------------
# 1. Clean slate
#    The ubuntu:24.04 image ships with a default "ubuntu" user at UID 1000.
#    Remove it so the lab accounts get exactly the IDs shown in the tutorial.
# -----------------------------------------------------------------------------
RUN userdel -r ubuntu 2>/dev/null || true

# -----------------------------------------------------------------------------
# 2. Create the Acme Corp accounts
#    Order matters: useradd appends to /etc/passwd, so this is the order
#    students will see them in.
#
#    user        UID   GID   notes
#    alice       1000  1002  Developer
#    bob         1001  1003  Accountant
#    carol       1002  1004  IT Administrator, member of the sudo group  <- target
#    svc_backup   999   999  service account, UID just under 1000,
#                            but has an interactive shell                 <- hidden target
#    student     1003  1005  the foothold the "attacker" starts from
# -----------------------------------------------------------------------------
RUN groupadd -g 999  svc_backup \
 && groupadd -g 1002 alice \
 && groupadd -g 1003 bob \
 && groupadd -g 1004 carol \
 && groupadd -g 1005 student \
 \
 && useradd -m -u 1000 -g alice -s /bin/bash \
        -c "Alice Nguyen,,,Developer" alice \
 && useradd -m -u 1001 -g bob -s /bin/bash \
        -c "Bob Martinez,,,Accountant" bob \
 && useradd -m -u 1002 -g carol -G sudo -s /bin/bash \
        -c "Carol Smith,,,IT Administrator" carol \
 && useradd -M -u 999 -g svc_backup -d /var/lib/backup -s /bin/bash \
        -c "Nightly backup job" svc_backup \
 && useradd -m -u 1003 -g student -s /bin/bash \
        -c "Student (you!)" student \
 \
 && mkdir -p /var/lib/backup \
 && chown svc_backup:svc_backup /var/lib/backup \
 && chmod 750 /var/lib/backup /home/alice /home/bob /home/carol /home/student

# -----------------------------------------------------------------------------
# 3. Give every account a real (but random, never-shown) password
#    This makes /etc/shadow look like a real server's, and it means nobody,
#    including the student, can "su" into another account. Discovery gives
#    you a target list, not the keys. root stays locked, as in the base image.
# -----------------------------------------------------------------------------
RUN for u in alice bob carol svc_backup student; do \
        echo "$u:$(head -c 24 /dev/urandom | base64)"; \
    done | chpasswd

# -----------------------------------------------------------------------------
# 4. Drop the student into the lab as the low-privilege "student" user
# -----------------------------------------------------------------------------
USER student
WORKDIR /home/student

# Keep the container running so students can "docker exec" into it.
CMD ["sleep", "infinity"]
