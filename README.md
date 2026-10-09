# Web tables

```

from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait, Select
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException
import re

driver = webdriver.Chrome()
wait = WebDriverWait(driver, 15)

driver.get("https://assertqa.com/practice/webtables")
driver.maximize_window()

# Wait for the employee table
wait.until(
    EC.presence_of_element_located((By.TAG_NAME, "table"))
)

table = driver.find_element(By.TAG_NAME, "table")


def get_headers():
    return [
        h.text.strip()
        for h in table.find_elements(By.XPATH, ".//th")
    ]


def get_rows():
    return table.find_elements(By.XPATH, ".//tbody/tr")


def get_row_values(row):
    return [
        cell.text.strip()
        for cell in row.find_elements(By.XPATH, "./td")
    ]


def print_result(test_case, status, message):
    print(f"\n{test_case} - {status}")
    print(message)


# Try to display all 15 employees on one page
for dropdown in driver.find_elements(By.TAG_NAME, "select"):
    try:
        select = Select(dropdown)
        options = [option.text.strip()
                   for option in select.options]

        if "15" in options:
            select.select_by_visible_text("15")
            break
    except Exception:
        pass

# TC01: Print all column headings
headers = get_headers()

print("\n========== TC01 ==========")
print("Column Headings:")

for heading in headers:
    print(heading)

print_result(
    "TC01",
    "PASS" if headers else "FAIL",
    "All headings displayed" if headers
    else "No headings found"
)


# TC02: Print the first data row
rows = get_rows()

print("\n========== TC02 ==========")

if rows:
    first_row = get_row_values(rows[0])
    print_result(
        "TC02",
        "PASS",
        "First employee record: " + " | ".join(first_row)
    )
else:
    print_result("TC02", "FAIL", "No employee records found")


# TC03: Print the last data row
rows = get_rows()

print("\n========== TC03 ==========")

if rows:
    last_row = get_row_values(rows[-1])
    print_result(
        "TC03",
        "PASS",
        "Last displayed employee record: "
        + " | ".join(last_row)
    )
else:
    print_result("TC03", "FAIL", "No employee records found")


# TC04: Search by last name
print("\n========== TC04 ==========")

search_box = wait.until(
    EC.visibility_of_element_located(
        (By.CSS_SELECTOR, "input[placeholder*='Search']")
    )
)

search_box.clear()
search_box.send_keys("Smith")

wait.until(
    lambda d: len(get_rows()) > 0
    or "0 of" in d.find_element(
        By.TAG_NAME, "body"
    ).text
)

matching_rows = [
    get_row_values(row)
    for row in get_rows()
    if len(get_row_values(row)) > 1
    and "smith" in get_row_values(row)[1].lower()
]

if matching_rows:
    for record in matching_rows:
        print_result(
            "TC04",
            "PASS",
            "Matching record: " + " | ".join(record)
        )
else:
    print_result(
        "TC04",
        "FAIL",
        "No employee with last name Smith found"
    )

# Clear search before the next tests
search_box.clear()
wait.until(lambda d: len(get_rows()) > 0)


# TC05: Extract all email addresses
print("\n========== TC05 ==========")

headers = get_headers()
email_index = next(
    (i for i, h in enumerate(headers)
     if h.lower() == "email"),
    None
)

emails = []

if email_index is not None:
    for row in get_rows():
        values = get_row_values(row)

        if len(values) > email_index:
            email = values[email_index]

            if re.fullmatch(
                r"[^@\s]+@[^@\s]+\.[^@\s]+", email
            ):
                emails.append(email)

for email in emails:
    print(email)

print_result(
    "TC05",
    "PASS" if emails else "FAIL",
    f"Total email addresses extracted: {len(emails)}"
)


# TC06: Find employee with highest Due amount
print("\n========== TC06 ==========")

headers = get_headers()
due_index = next(
    (i for i, h in enumerate(headers)
     if "due" in h.lower()),
    None
)

if due_index is None:
    print_result(
        "TC06",
        "FAIL",
        "The demo table has no Due amount column."
    )
else:
    due_records = []

    for row in get_rows():
        values = get_row_values(row)

        if len(values) > due_index:
            amount = re.sub(r"[^0-9.-]", "",
                            values[due_index])

            try:
                due_records.append(
                    (float(amount), values)
                )
            except ValueError:
                pass

    if due_records:
        highest = max(due_records, key=lambda x: x[0])

        print_result(
            "TC06",
            "PASS",
            f"Highest Due: {highest[0]}\n"
            f"Employee record: {' | '.join(highest[1])}"
        )
    else:
        print_result("TC06", "FAIL", "No valid Due amounts found")


# TC07: Verify a website link exists in the table
print("\n========== TC07 ==========")

website_index = next(
    (i for i, h in enumerate(headers)
     if "website" in h.lower()
     or "url" in h.lower()),
    None
)

links = table.find_elements(By.XPATH, ".//a[@href]")

if website_index is not None or links:
    print_result(
        "TC07",
        "PASS",
        f"Found {len(links)} link(s) in the table."
    )
else:
    print_result(
        "TC07",
        "FAIL",
        "No website/URL column or links found in the table."
    )


# TC08: Count data rows without the header
print("\n========== TC08 ==========")

data_rows = get_rows()
row_count = len(data_rows)

print_result(
    "TC08",
    "PASS" if row_count > 0 else "FAIL",
    f"Number of displayed data rows: {row_count}"
)


print("\n========== TEST EXECUTION COMPLETED ==========")

input("Press Enter to close the browser...")
driver.quit()

```
