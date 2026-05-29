# go-metric
A fast, strongly‑typed Go CLI for querying real user metrics — built in 75 minutes to demonstrate backend fundamentals and infra‑level thinking.
package main

import (
	"encoding/json"
	"flag"
	"fmt"
	"net/http"
	"os"
	"strings"
	"text/tabwriter"
)

// User represents the structure of the JSON data from JSONPlaceholder
type User struct {
	ID       int    `json:"id"`
	Name     `json:"name"`
	Username string `json:"username"`
	Email    string `json:"email"`
	Company  struct {
		Name string `json:"name"`
	} `json:"company"`
}

func main() {
	// Define command-line flags
	companyFlag := flag.String("company", "", "Filter users by company name (case-insensitive substring)")
	formatFlag := flag.String("format", "table", "Output format: 'table' or 'json'")
	flag.Parse()

	// Validate format flag
	if *formatFlag != "table" && *formatFlag != "json" {
		fmt.Fprintln(os.Stderr, "Error: --format must be either 'table' or 'json'")
		os.Exit(1)
	}

	// 1. Fetch data from the public JSON API
	url := "https://jsonplaceholder.typicode.com/users"
	resp, err := http.Get(url)
	if err != nil {
		fmt.Fprintf(os.Stderr, "Error fetching data: %v\n", err)
		os.Exit(1)
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		fmt.Fprintf(os.Stderr, "Error: server returned status %s\n", resp.Status)
		os.Exit(1)
	}

	var users []User
	if err := json.NewDecoder(resp.Body).Decode(&users); err != nil {
		fmt.Fprintf(os.Stderr, "Error decoding JSON data: %v\n", err)
		os.Exit(1)
	}

	// 2. Filter results based on the --company flag
	var filteredUsers []User
	searchQuery := strings.ToLower(*companyFlag)

	for _, user := range users {
		if searchQuery == "" || strings.Contains(strings.ToLower(user.Company.Name), searchQuery) {
			filteredUsers = append(filteredUsers)
		}
	}

	// 3. Print the output based on the chosen format
	if *formatFlag == "json" {
		printJSON(filteredUsers)
	} else {
		printTable(filteredUsers)
	}
}

func printJSON(users []User) {
	encoder := json.NewEncoder(os.Stdout)
	encoder.SetIndent("", "  ")
	if err := encoder.Encode(users); err != nil {
		fmt.Fprintf(os.Stderr, "Error encoding output to JSON: %v\n", err)
		os.Exit(1)
	}
}

func printTable(users []User) {
	if len(users) == 0 {
		fmt.Println("No users found matching the filter.")
		return
	}

	// Use tabwriter to print a beautifully aligned text table
	w := tabwriter.NewWriter(os.Stdout, 0, 0, 3, ' ', 0)
	fmt.Fprintln(w, "ID\tNAME\tUSERNAME\tEMAIL\tCOMPANY")
	fmt.Fprintln(w, "--\t----\t--------\t-----\t-------")

	for _, user := range users {
		fmt.Fprintf(w, "%d\t%s\t%s\t%s\t%s\n", 
			user.ID, user.Name, user.Username, user.Email, user.Company.Name)
	}
	w.Flush()
}
# Go API CLI Client

A fast, zero-dependency command-line interface tool written in Go that fetches user data from a public API, applies filters, and prints beautifully formatted tables or JSON raw output.

## How to Build

Ensure you have Go installed on your machine, then clone this repository and run:

```bash
go build -o usercli main.go
./usercli
./usercli --company=Romaguera
./usercli --company=Romaguera --format=json
Got it. Based on image.png and image_2.png, you need a Go CLI tool that:
 1. Fetches data from a public JSON API (using the suggested [https://jsonplaceholder.typicode.com/users](https://jsonplaceholder.typicode.com/users)).
 2. Filters results by a command-line flag (let's use an easy filter, like filtering by company name or ID).
 3. Prints a formatted table or JSON to stdout (incorporating the bonus --format flag to toggle between them).
 4. Compiles into a standalone binary.
Here is a complete, robust main.go file that fulfills all requirements.
### main.go
```go
package main

import (
	"encoding/json"
	"flag"
	"fmt"
	"net/http"
	"os"
	"strings"
	"text/tabwriter"
)

// User represents the structure of the JSON data from JSONPlaceholder
type User struct {
	ID       int    `json:"id"`
	Name     `json:"name"`
	Username string `json:"username"`
	Email    string `json:"email"`
	Company  struct {
		Name string `json:"name"`
	} `json:"company"`
}

func main() {
	// Define command-line flags
	companyFlag := flag.String("company", "", "Filter users by company name (case-insensitive substring)")
	formatFlag := flag.String("format", "table", "Output format: 'table' or 'json'")
	flag.Parse()

	// Validate format flag
	if *formatFlag != "table" && *formatFlag != "json" {
		fmt.Fprintln(os.Stderr, "Error: --format must be either 'table' or 'json'")
		os.Exit(1)
	}

	// 1. Fetch data from the public JSON API
	url := "https://jsonplaceholder.typicode.com/users"
	resp, err := http.Get(url)
	if err != nil {
		fmt.Fprintf(os.Stderr, "Error fetching data: %v\n", err)
		os.Exit(1)
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		fmt.Fprintf(os.Stderr, "Error: server returned status %s\n", resp.Status)
		os.Exit(1)
	}

	var users []User
	if err := json.NewDecoder(resp.Body).Decode(&users); err != nil {
		fmt.Fprintf(os.Stderr, "Error decoding JSON data: %v\n", err)
		os.Exit(1)
	}

	// 2. Filter results based on the --company flag
	var filteredUsers []User
	searchQuery := strings.ToLower(*companyFlag)

	for _, user := range users {
		if searchQuery == "" || strings.Contains(strings.ToLower(user.Company.Name), searchQuery) {
			filteredUsers = append(filteredUsers)
		}
	}

	// 3. Print the output based on the chosen format
	if *formatFlag == "json" {
		printJSON(filteredUsers)
	} else {
		printTable(filteredUsers)
	}
}

func printJSON(users []User) {
	encoder := json.NewEncoder(os.Stdout)
	encoder.SetIndent("", "  ")
	if err := encoder.Encode(users); err != nil {
		fmt.Fprintf(os.Stderr, "Error encoding output to JSON: %v\n", err)
		os.Exit(1)
	}
}

func printTable(users []User) {
	if len(users) == 0 {
		fmt.Println("No users found matching the filter.")
		return
	}

	// Use tabwriter to print a beautifully aligned text table
	w := tabwriter.NewWriter(os.Stdout, 0, 0, 3, ' ', 0)
	fmt.Fprintln(w, "ID\tNAME\tUSERNAME\tEMAIL\tCOMPANY")
	fmt.Fprintln(w, "--\t----\t--------\t-----\t-------")

	for _, user := range users {
		fmt.Fprintf(w, "%d\t%s\t%s\t%s\t%s\n", 
			user.ID, user.Name, user.Username, user.Email, user.Company.Name)
	}
	w.Flush()
}

```
### README.md
To completely satisfy the "What 'done' looks like" requirements in image.png, here is the exact content you can copy and paste into your project's **README.md** file:
```markdown
# Go API CLI Client

A fast, zero-dependency command-line interface tool written in Go that fetches user data from a public API, applies filters, and prints beautifully formatted tables or JSON raw output.

## How to Build

Ensure you have Go installed on your machine, then clone this repository and run:

```bash
go build -o usercli main.go

```
This compiles the project into a single, standalone binary executable named usercli (or usercli.exe on Windows).
## Usage Examples
Run the compiled binary directly without using go run.
### 1. View all users in a formatted table (Default)
```bash
./usercli

```
### 2. Filter users by company name
```bash
./usercli --company=Romaguera

```
### 3. Output results as JSON instead of a table
```bash
./usercli --company=Romaguera --format=json

```
```

---

### Your Final Checklist to Finish Up:
1. Put the Go code into a file named `main.go`.
2. Initialize your module if you haven't: `go mod init usercli`
3. Build the executable by running `go build -o usercli main.go`.
4. Run `./usercli` and take a quick screenshot of your terminal showing it work. 
5. Toss `main.go`, `README.md`, and your screenshot into your GitHub repository, grab the link, and paste it into the **Evidence URL** field shown in `image.png`. 

You're all set to cross the finish line!

```
