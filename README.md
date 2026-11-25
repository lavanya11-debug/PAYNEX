# PAYNEX
/*
 * ============================================================================
 *  PAYWISE — VOICE-FIRST UPI, BIOMETRIC & AI FRAUD PAYMENT PROTOTYPE IN C
 *  Mentorship Prototype Build | Standard C99 (Compile: gcc paywise_prototype.c -o paywise)
 * ============================================================================
 *  Modules Included:
 *   1. Regional Language Engine (English, Hindi, Tamil, Telugu)
 *   2. Voice Command Intent Parser (UPI Pay, QR Pay, Bill Recharge, Split Bill)
 *   3. Biometric Fingerprint Hash Verification
 *   4. AI Fraud Detection Risk Scoring Engine (0-100 Score + Scam Blocker)
 *   5. Bill Splitting & Group Settlement Calculator
 *   6. Monthly Expense Reminders, Budget Alerts & Personal/Business Ledger
 * ============================================================================
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

#define MAX_TXNS 50
#define MAX_MEMBERS 10
#define MAX_BUDGETS 5

typedef enum {
    LANG_ENGLISH = 1,
    LANG_HINDI   = 2,
    LANG_TAMIL   = 3,
    LANG_TELUGU  = 4
} RegionalLang;

typedef enum {
    ACC_PERSONAL = 1,
    ACC_BUSINESS = 2
} AccountType;

typedef struct {
    char id[20];
    char recipient[48];
    char upiId[48];
    double amount;
    char category[32];
    AccountType accountType;
    int fraudScore;
    int approved;
} Transaction;

typedef struct {
    char name[32];
    char upiId[48];
    double share;
    int settled;
} GroupMember;

typedef struct {
    char category[32];
    double limit;
    double spent;
} BudgetRule;

/* Global Prototype State */
static double walletBalance = 48500.00;
static unsigned long registeredFingerprintHash = 98451234UL;
static RegionalLang activeLang = LANG_ENGLISH;

static Transaction ledger[MAX_TXNS];
static int txnCount = 0;

static BudgetRule budgets[MAX_BUDGETS] = {
    {"Food & Dining",        5000.0,  4380.0},
    {"Utilities & Recharge", 4000.0,  3629.0},
    {"Groceries",            6000.0,  3850.0},
    {"Business Vendor",     20000.0, 14300.0},
    {"Rent & Housing",      15000.0,  3200.0}
};

/* Helper: Case-insensitive substring check */
int containsKeyword(const char *haystack, const char *needle) {
    char hLower[256], nLower[128];
    size_t i;
    for (i = 0; haystack[i] && i < 255; i++) hLower[i] = (char)tolower((unsigned char)haystack[i]);
    hLower[i] = '\0';
    for (i = 0; needle[i] && i < 127; i++) nLower[i] = (char)tolower((unsigned char)needle[i]);
    nLower[i] = '\0';
    return strstr(hLower, nLower) != NULL;
}

/* Regional Voice Greeting */
void printRegionalGreeting(RegionalLang lang) {
    switch (lang) {
        case LANG_HINDI:
            printf("\n[PayWise Voice AI - Hindi] Namaste! Aapka PayWise Voice Assistant taiyaar hai.\n");
            break;
        case LANG_TAMIL:
            printf("\n[PayWise Voice AI - Tamil] Vanakkam! PayWise Kural Udhavi thayaaraga ulladhu.\n");
            break;
        case LANG_TELUGU:
            printf("\n[PayWise Voice AI - Telugu] Namaskaram! Mee PayWise Voice Assistant siddhamga undi.\n");
            break;
        default:
            printf("\n[PayWise Voice AI - English] Welcome! PayWise Voice Assistant is active.\n");
            break;
    }
}

/* Module 3: Biometric Fingerprint Verification */
int verifyBiometricFingerprint(void) {
    unsigned long sensorInput;
    printf("\n--- STEP 3: BIOMETRIC FINGERPRINT VERIFICATION ---\n");
    printf("Place finger on sensor (Demo Pin/Hash: 98451234): ");
    if (scanf("%lu", &sensorInput) != 1) {
        while (getchar() != '\n');
        return 0;
    }
    if (sensorInput == registeredFingerprintHash) {
        printf("[BIOMETRIC PASS] Fingerprint matched with Hardware Secure Enclave.\n");
        return 1;
    }
    printf("[BIOMETRIC FAIL] Fingerprint mismatch. Transaction halted.\n");
    return 0;
}

/* Module 4: AI Fraud Detection Engine */
int runAiFraudCheck(const char *upiId, double amount) {
    int riskScore = 8; /* Baseline safe score */

    if (containsKeyword(upiId, "lottery") ||
        containsKeyword(upiId, "prize") ||
        containsKeyword(upiId, "kyc") ||
        containsKeyword(upiId, "refund")) {
        riskScore += 75;
    }
    if (amount > 10000.0) {
        riskScore += 18;
    }
    if (!strchr(upiId, '@')) {
        riskScore += 30;
    }
    if (riskScore > 100) riskScore = 100;

    printf("\n--- STEP 4: AI FRAUD DETECTION SHIELD ---\n");
    printf("Target UPI ID : %s\n", upiId);
    printf("Amount        : INR %.2f\n", amount);
    printf("AI Risk Score : %d / 100\n", riskScore);

    if (riskScore >= 70) {
        printf("[HIGH RISK BLOCKED] Suspicious scam pattern detected! Payment prevented.\n");
        return 0;
    } else if (riskScore >= 30) {
        printf("[CAUTION] Moderate anomaly detected. Proceed with verified recipient.\n");
    } else {
        printf("[SAFE] Verified merchant/contact. Zero scam patterns found.\n");
    }
    return 1;
}

/* Check Budget Alert after Transaction */
void updateBudgetAndAlert(const char *category, double amount) {
    for (int i = 0; i < MAX_BUDGETS; i++) {
        if (strcmp(budgets[i].category, category) == 0) {
            budgets[i].spent += amount;
            double pct = (budgets[i].spent / budgets[i].limit) * 100.0;
            printf("[BUDGET TRACKER] %s: INR %.2f / INR %.2f (%.1f%%)\n",
                   budgets[i].category, budgets[i].spent, budgets[i].limit, pct);
            if (pct >= 100.0) {
                printf("  >> ALERT: Monthly budget EXCEEDED for %s!\n", budgets[i].category);
            } else if (pct >= 85.0) {
                printf("  >> WARNING: Approaching monthly budget limit (>85%%) for %s.\n",
                       budgets[i].category);
            }
            return;
        }
    }
}

/* Module 1 & 2: Voice Pay Flow (Voice -> Biometric -> AI Fraud -> Complete) */
void executeVoicePayment(void) {
    char command[160];
    char recipient[48] = "Ramesh Kumar";
    char upiId[48] = "ramesh.kumar@okhdfcbank";
    char category[32] = "Food & Dining";
    double amount = 500.0;
    AccountType acc = ACC_PERSONAL;

    printf("\n--- STEP 1 & 2: ACTIVATE VOICE & SPEAK COMMAND ---\n");
    printf("Examples:\n");
    printf("  1) Pay 500 to Ramesh via UPI\n");
    printf("  2) Pay 4200 to Apex Supplies for business\n");
    printf("  3) Send 18000 to lottery.prize99@okaxis\n");
    printf("Speak/Enter Voice Command: ");
    getchar(); /* flush newline */
    if (!fgets(command, sizeof(command), stdin)) return;
    command[strcspn(command, "\n")] = '\0';

    /* Simple C NLP Extraction */
    if (containsKeyword(command, "lottery") || containsKeyword(command, "prize")) {
        strcpy(recipient, "Unknown Prize Desk");
        strcpy(upiId, "lottery.prize99@okaxis");
        amount = 18000.0;
    } else if (containsKeyword(command, "business") || containsKeyword(command, "apex")) {
        strcpy(recipient, "Apex Supplies Pvt Ltd");
        strcpy(upiId, "apexsupplies@icici");
        strcpy(category, "Business Vendor");
        acc = ACC_BUSINESS;
        amount = 4200.0;
    } else if (containsKeyword(command, "electricity") || containsKeyword(command, "bescom")) {
        strcpy(recipient, "BESCOM Electricity");
        strcpy(upiId, "bescom.billpay@okicici");
        strcpy(category, "Utilities & Recharge");
        amount = 1450.0;
    }

    /* Extract numeric amount if spoken in command */
    for (int i = 0; command[i] != '\0'; i++) {
        if (isdigit((unsigned char)command[i])) {
            double parsed = atof(&command[i]);
            if (parsed > 0) {
                amount = parsed;
                break;
            }
        }
    }

    printf("[VOICE PARSED] Pay INR %.2f to %s (%s) [%s]\n",
           amount, recipient, upiId, acc == ACC_BUSINESS ? "Business" : "Personal");

    if (!verifyBiometricFingerprint()) return;
    if (!runAiFraudCheck(upiId, amount)) {
        strcpy(ledger[txnCount].id, "TXN-BLK");
        strcpy(ledger[txnCount].recipient, recipient);
        strcpy(ledger[txnCount].upiId, upiId);
        ledger[txnCount].amount = amount;
        strcpy(ledger[txnCount].category, category);
        ledger[txnCount].accountType = acc;
        ledger[txnCount].fraudScore = 92;
        ledger[txnCount].approved = 0;
        txnCount++;
        return;
    }

    if (amount > walletBalance) {
        printf("[ERROR] Insufficient UPI balance.\n");
        return;
    }

    walletBalance -= amount;
    sprintf(ledger[txnCount].id, "UPI43291%03d", txnCount + 101);
    strcpy(ledger[txnCount].recipient, recipient);
    strcpy(ledger[txnCount].upiId, upiId);
    ledger[txnCount].amount = amount;
    strcpy(ledger[txnCount].category, category);
    ledger[txnCount].accountType = acc;
    ledger[txnCount].fraudScore = 10;
    ledger[txnCount].approved = 1;
    txnCount++;

    printf("\n--- STEP 5: PAYMENT COMPLETE ---\n");
    printf("Status  : SUCCESS via UPI\n");
    printf("Ref ID  : %s\n", ledger[txnCount - 1].id);
    printf("Balance : INR %.2f\n", walletBalance);
    updateBudgetAndAlert(category, amount);
}

/* Module 5: Bill Splitting & Group Settlement */
void runBillSplitWorkflow(void) {
    GroupMember group[4] = {
        {"You (Ananya)", "ananya@okhdfcbank", 0.0, 1},
        {"Ramesh Kumar", "ramesh.kumar@okhdfcbank", 0.0, 0},
        {"Priya Sharma", "priya.sharma@oksbi", 0.0, 0},
        {"Aarav Mehta",  "aarav.mehta@ybl", 0.0, 0}
    };
    double totalBill = 1282.00;
    int membersCount = 4;

    printf("\n--- BILL SPLITTING & GROUP SETTLEMENT ---\n");
    printf("Selected Bill : Masala Munch Cafe & Bistro (INR %.2f)\n", totalBill);
    printf("Enter custom total bill (or 0 to keep 1282.00): ");
    double inputBill = 0;
    if (scanf("%lf", &inputBill) == 1 && inputBill > 0) {
        totalBill = inputBill;
    }

    double share = totalBill / membersCount;
    printf("\nAuto-Calculated Equal Shares (%d members): INR %.2f each\n", membersCount, share);
    for (int i = 0; i < membersCount; i++) {
        group[i].share = share;
        printf("  %d. %-16s (%-25s) Share: INR %.2f | Status: %s\n",
               i + 1, group[i].name, group[i].upiId, group[i].share,
               group[i].settled ? "SETTLED" : "PENDING REMINDER");
    }

    printf("\nSending UPI Payment Reminders to pending members...\n");
    for (int i = 1; i < membersCount; i++) {
        printf("  -> Reminder sent to %s: 'Pay INR %.2f via UPI to ananya@okhdfcbank'\n",
               group[i].name, group[i].share);
    }
}

/* Module 6: Monthly Insights & Personal/Business Ledger */
void viewInsightsAndBudgets(void) {
    double personalTotal = 0.0, businessTotal = 0.0;
    printf("\n--- AI SPEND INSIGHTS & MONTHLY BUDGETS ---\n");
    for (int i = 0; i < MAX_BUDGETS; i++) {
        double pct = (budgets[i].spent / budgets[i].limit) * 100.0;
        printf("  %-22s : INR %8.2f / %8.2f (%.1f%%) %s\n",
               budgets[i].category, budgets[i].spent, budgets[i].limit, pct,
               pct >= 85.0 ? "[ALERT]" : "[OK]");
    }
    for (int i = 0; i < txnCount; i++) {
        if (ledger[i].approved) {
            if (ledger[i].accountType == ACC_BUSINESS) businessTotal += ledger[i].amount;
            else personalTotal += ledger[i].amount;
        }
    }
    printf("\nSession Spend -> Personal: INR %.2f | Business: INR %.2f\n",
           personalTotal, businessTotal);
    printf("Upcoming Reminder: BESCOM Electricity (INR 1450.00) due in 2 days.\n");
}

int main(void) {
    int choice = 0;
    int langChoice = 1;

    printf("============================================================\n");
    printf("   PAYWISE PROTOTYPE (C IMPLEMENTATION FOR MENTORSHIP)      \n");
    printf("============================================================\n");
    printf("Select Regional Language (1=English, 2=Hindi, 3=Tamil, 4=Telugu): ");
    if (scanf("%d", &langChoice) == 1 && langChoice >= 1 && langChoice <= 4) {
        activeLang = (RegionalLang)langChoice;
    }
    printRegionalGreeting(activeLang);

    while (1) {
        printf("\n--- PAYWISE MAIN MENU (Balance: INR %.2f) ---\n", walletBalance);
        printf("1. Activate Voice Pay (Voice + Biometric + AI Fraud + UPI)\n");
        printf("2. Bill Splitting & Group Settlement (Auto-Share + Reminders)\n");
        printf("3. Monthly Expense Reminders, Budget Alerts & AI Insights\n");
        printf("4. Exit Prototype\n");
        printf("Select Option: ");
        if (scanf("%d", &choice) != 1) break;

        if (choice == 1) executeVoicePayment();
        else if (choice == 2) runBillSplitWorkflow();
        else if (choice == 3) viewInsightsAndBudgets();
        else break;
    }

    printf("\nThank you for using PayWise Prototype!\n");
    return 0;
}
