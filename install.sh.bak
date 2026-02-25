#!/bin/bash
# SniffCat fail2ban Integration - Installer
# https://github.com/Rexikon/SniffCat-fail2ban
#
# Usage:
#   bash <(curl -fsSL https://raw.githubusercontent.com/Rexikon/SniffCat-fail2ban/main/install.sh)
#
# SPDX-License-Identifier: GPL-3.0-or-later

set -euo pipefail

# --- Configuration ---
INSTALL_DIR="/opt/sniffcat"
CONFIG_FILE="${INSTALL_DIR}/sniffcat.conf"
SCRIPT_NAME="fail2ban.sh"
ACTION_NAME="sniffcat-action.conf"
ACTION_DEST="/etc/fail2ban/action.d/sniffcat.conf"
REPO_URL="https://raw.githubusercontent.com/Rexikon/SniffCat-fail2ban/main"
LOG_FILE="/var/log/sniffcat.log"

# --- Colors ---
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
BOLD='\033[1m'
NC='\033[0m'

# --- Functions ---
info()    { echo -e "${BLUE}[INFO]${NC} $*"; }
success() { echo -e "${GREEN}[OK]${NC} $*"; }
warn()    { echo -e "${YELLOW}[WARN]${NC} $*"; }
error()   { echo -e "${RED}[ERROR]${NC} $*"; exit 1; }

banner() {
    echo ""
    echo -e "${BOLD}"
    echo "  ╔═══════════════════════════════════════╗"
    echo "  ║    SniffCat fail2ban Integration       ║"
    echo "  ║            Installer v1.0              ║"
    echo "  ╚═══════════════════════════════════════╝"
    echo -e "${NC}"
}

# --- Banner ---
banner

# --- Pre-checks ---

# Root check
if [[ $EUID -ne 0 ]]; then
    error "This script must be run as root. Use: sudo bash install.sh"
fi

# Check for curl
if ! command -v curl &>/dev/null; then
    error "curl is required but not installed."
fi

# Check for fail2ban
if ! command -v fail2ban-client &>/dev/null; then
    warn "fail2ban does not appear to be installed on this server."
    read -rp "Continue anyway? [y/N]: " confirm
    [[ "$confirm" =~ ^[Yy]$ ]] || exit 0
fi

# Check fail2ban action.d directory
if [[ ! -d "/etc/fail2ban/action.d" ]]; then
    warn "Directory /etc/fail2ban/action.d not found."
    read -rp "Continue anyway? [y/N]: " confirm
    [[ "$confirm" =~ ^[Yy]$ ]] || exit 0
fi

# Check for existing installation
if [[ -f "${INSTALL_DIR}/${SCRIPT_NAME}" ]]; then
    warn "Existing installation detected at ${INSTALL_DIR}"
    read -rp "Overwrite? [y/N]: " confirm
    [[ "$confirm" =~ ^[Yy]$ ]] || exit 0
fi

# --- Token Input ---
echo ""
info "You need a SniffCat API token to continue."
info "Get your token at: ${BOLD}https://sniffcat.com${NC}"
echo ""

read -rp "$(echo -e "${YELLOW}Enter your SniffCat API token: ${NC}")" TOKEN

if [[ -z "$TOKEN" ]]; then
    error "Token cannot be empty."
fi

if [[ ${#TOKEN} -lt 10 ]]; then
    warn "Token seems too short. Are you sure it's correct?"
    read -rp "Continue? [y/N]: " confirm
    [[ "$confirm" =~ ^[Yy]$ ]] || exit 0
fi

# --- Installation ---
echo ""
info "Installing SniffCat fail2ban integration..."

# Create install directory
mkdir -p "$INSTALL_DIR"
success "Created directory: ${INSTALL_DIR}"

# Download reporting script
info "Downloading ${SCRIPT_NAME}..."
if curl -fsSL "${REPO_URL}/${SCRIPT_NAME}" -o "${INSTALL_DIR}/${SCRIPT_NAME}"; then
    success "Downloaded: ${INSTALL_DIR}/${SCRIPT_NAME}"
else
    error "Failed to download ${SCRIPT_NAME} from ${REPO_URL}"
fi

# Download and install fail2ban action
info "Installing fail2ban action..."
if curl -fsSL "${REPO_URL}/${ACTION_NAME}" -o "${ACTION_DEST}"; then
    success "Installed action: ${ACTION_DEST}"
else
    error "Failed to download fail2ban action from ${REPO_URL}"
fi

# Create config file
cat > "$CONFIG_FILE" <<EOF
# SniffCat Configuration
# https://sniffcat.com
#
# API Token for authentication
SNIFFCAT_TOKEN="${TOKEN}"
EOF
success "Created config: ${CONFIG_FILE}"

# Set permissions
chmod 755 "${INSTALL_DIR}/${SCRIPT_NAME}"
chmod 600 "$CONFIG_FILE"
chmod 644 "$ACTION_DEST"
success "Set permissions (script: 755, config: 600, action: 644)"

# Create log file
touch "$LOG_FILE"
chmod 640 "$LOG_FILE"
success "Created log file: ${LOG_FILE}"

# --- Verify installation ---
echo ""
info "Verifying installation..."

if [[ -x "${INSTALL_DIR}/${SCRIPT_NAME}" ]] && [[ -f "$CONFIG_FILE" ]] && [[ -f "$ACTION_DEST" ]]; then
    success "Installation completed successfully!"
else
    error "Installation verification failed."
fi

# --- Next steps ---
echo ""
echo -e "${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
echo -e "${BOLD}  Next Steps${NC}"
echo -e "${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
echo ""
echo -e "  Add ${GREEN}sniffcat${NC} action to your jails in ${BOLD}/etc/fail2ban/jail.local${NC}"
echo ""
echo -e "  ${BOLD}Option A — Add to a specific jail:${NC}"
echo ""
echo -e "    ${BLUE}[sshd]${NC}"
echo -e "    ${BLUE}enabled = true${NC}"
echo -e "    ${BLUE}action = %(action_)s${NC}"
echo -e "    ${BLUE}         sniffcat${NC}"
echo ""
echo -e "  ${BOLD}Option B — Add to all jails globally:${NC}"
echo ""
echo -e "    ${BLUE}[DEFAULT]${NC}"
echo -e "    ${BLUE}action = %(action_)s${NC}"
echo -e "    ${BLUE}         sniffcat${NC}"
echo ""
echo -e "  Then restart fail2ban:"
echo -e "    ${GREEN}sudo systemctl restart fail2ban${NC}"
echo ""
echo -e "  ${BLUE}Logs:${NC}   ${LOG_FILE}"
echo -e "  ${BLUE}Config:${NC} ${CONFIG_FILE}"
echo -e "  ${BLUE}Action:${NC} ${ACTION_DEST}"
echo ""
