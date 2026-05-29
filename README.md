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
