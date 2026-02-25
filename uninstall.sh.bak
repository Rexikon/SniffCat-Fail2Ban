#!/bin/bash
# SniffCat fail2ban Integration - Uninstaller
# https://github.com/Rexikon/SniffCat-fail2ban
#
# Usage:
#   bash <(curl -fsSL https://raw.githubusercontent.com/Rexikon/SniffCat-fail2ban/main/uninstall.sh)
#
# SPDX-License-Identifier: GPL-3.0-or-later

set -euo pipefail

INSTALL_DIR="/opt/sniffcat"
ACTION_FILE="/etc/fail2ban/action.d/sniffcat.conf"
LOG_FILE="/var/log/sniffcat.log"

# --- Colors ---
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BOLD='\033[1m'
NC='\033[0m'

# --- Root check ---
if [[ $EUID -ne 0 ]]; then
    echo -e "${RED}[ERROR]${NC} This script must be run as root."
    exit 1
fi

# --- Confirm ---
echo ""
echo -e "${BOLD}SniffCat fail2ban Integration - Uninstaller${NC}"
echo ""
echo -e "${YELLOW}The following will be removed:${NC}"
echo "  - ${INSTALL_DIR}/ (directory and all contents)"
echo "  - ${ACTION_FILE}"
echo "  - ${LOG_FILE}"
echo ""
read -rp "$(echo -e "${RED}Are you sure you want to uninstall? [y/N]: ${NC}")" confirm
[[ "$confirm" =~ ^[Yy]$ ]] || exit 0

# --- Remove files ---
if [[ -d "$INSTALL_DIR" ]]; then
    rm -rf "$INSTALL_DIR"
    echo -e "${GREEN}[OK]${NC} Removed ${INSTALL_DIR}"
else
    echo -e "${YELLOW}[SKIP]${NC} ${INSTALL_DIR} not found"
fi

if [[ -f "$ACTION_FILE" ]]; then
    rm -f "$ACTION_FILE"
    echo -e "${GREEN}[OK]${NC} Removed ${ACTION_FILE}"
else
    echo -e "${YELLOW}[SKIP]${NC} ${ACTION_FILE} not found"
fi

if [[ -f "$LOG_FILE" ]]; then
    rm -f "$LOG_FILE"
    echo -e "${GREEN}[OK]${NC} Removed ${LOG_FILE}"
else
    echo -e "${YELLOW}[SKIP]${NC} ${LOG_FILE} not found"
fi

echo ""
echo -e "${GREEN}Uninstallation complete.${NC}"
echo -e "${YELLOW}Remember to:${NC}"
echo -e "  1. Remove ${BOLD}sniffcat${NC} action from your jails in /etc/fail2ban/jail.local"
echo -e "  2. Restart fail2ban: ${BOLD}sudo systemctl restart fail2ban${NC}"
echo ""
