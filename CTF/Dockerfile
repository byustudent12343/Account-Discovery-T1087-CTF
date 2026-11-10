# =============================================================================
#  Acme Corp "acme-db02" - CTF for MITRE ATT&CK T1087.001
#  (Account Discovery: Local Account)
#
#  Flag format: CYBER266{A_B_C}
#  Every account's comment field holds a "tag=" word. Most are decoys.
#  Students must use account discovery to work out which three accounts
#  (A, B, C) matter, then join their tags. See README.md for the challenge.
#
#  Harmless training target: fake accounts only, no services, no exploits.
# =============================================================================
FROM ubuntu:24.04

LABEL description="T1087.001 Local Account Discovery CTF (Acme Corp acme-db02)"

# Remove the base image's default "ubuntu" user (UID 1000).
RUN userdel -r ubuntu 2>/dev/null || true

# -----------------------------------------------------------------------------
# Service accounts (UID < 1000). All but one are locked down with nologin
# or /bin/false. The root-equivalent account is slipped in among them so it
# blends into the noise in /etc/passwd.
# -----------------------------------------------------------------------------
RUN useradd -r -U -u 990 -M -d /var/www       -s /usr/sbin/nologin \
        -c "Web app runner,,,,tag=w3b"                 svc_web \
 && useradd -r -U -u 991 -M -d /var/lib/mon    -s /usr/sbin/nologin \
        -c "Monitoring agent,,,,tag=m0n1t0r"           svc_mon \
 && useradd -r -U -u 992 -M -d /var/lib/ftp    -s /bin/false \
        -c "Old FTP sync - disabled,,,,tag=0ld"        svc_legacy \
 && useradd -o -u 0 -g 0 -M -d /var/lib/sysupdate -s /bin/bash \
        -c "System update helper,,,,tag=us3rs"         sysupdate \
 && useradd -r -U -u 993 -M -d /var/lib/deploy -s /usr/sbin/nologin \
        -c "Deploy hook,,,,tag=d3pl0y"                 svc_deploy \
 && useradd -r -U -u 994 -M -d /var/spool/mail -s /usr/sbin/nologin \
        -c "Mail relay,,,,tag=m41l"                    svc_mail \
 && mkdir -p /var/lib/mon /var/lib/ftp /var/lib/sysupdate /var/lib/deploy \
 && chmod 700 /var/lib/sysupdate

# -----------------------------------------------------------------------------
# Employees (UID >= 1000), plus one service account hiding among them.
#
# Twists that stop the tutorial's exact commands from solving the CTF:
#  - eli's PRIMARY group is sudo (-g sudo). /etc/group only lists
#    supplementary members, so "getent group sudo" shows nobody at all.
#  - svc_ci is a service account that was created like a normal user, so it
#    got a UID above 1000. A "UID < 1000" filter misses it.
#  - dkim's "IT Administrator" title is a lie, and the sudo-requests group
#    catches anyone who just runs "grep sudo".
# -----------------------------------------------------------------------------
RUN groupadd developers \
 && groupadd sudo-requests \
 && useradd -m -U -s /bin/bash -G adm,sudo-requests \
        -c "Dana Kim,,,IT Administrator,tag=r00t"      dkim \
 && useradd -m -g sudo -s /bin/bash \
        -c "Eli Brooks,,,Summer Intern,tag=kn0w"       eli \
 && useradd -m -U -s /bin/bash \
        -c "Farah Patel,,,Accountant,tag=s3cur3"       fpatel \
 && useradd -m -U -s /bin/bash -G developers \
        -c "Grace Wu,,,Developer,tag=p4ssw0rd"         gwu \
 && useradd -U -M -d /opt/ci-runner -s /bin/bash \
        -c "CI pipeline runner,,,,tag=y0ur"            svc_ci \
 && mkdir -p /opt/ci-runner && chown svc_ci:svc_ci /opt/ci-runner \
 && useradd -m -U -s /bin/bash \
        -c "Hank Jones,,,Sales,tag=h4ck3r"             hjones \
 && useradd -m -U -s /bin/bash \
        -c "Student (you!),,,,tag=st4rt"               student \
 && chmod 750 /home/*

# Random, never-shown passwords for everyone, so nobody can su into the
# accounts they discover. root stays locked.
RUN for u in svc_ci sysupdate dkim eli fpatel gwu hjones student; do \
        echo "$u:$(head -c 24 /dev/urandom | base64)"; \
    done | chpasswd

# Flag checker. It stores only a SHA-256 hash, not the flag.
COPY checkflag /usr/local/bin/checkflag
RUN chmod 755 /usr/local/bin/checkflag

USER student
WORKDIR /home/student
CMD ["sleep", "infinity"]
