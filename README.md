#pragma comment(lib,"wininet.lib") /*This links wininet lib only works on Visual Studio VC++ */
#include<iostream>
#include<cstring>
#include<windows.h>
#include<wininet.h>
#include <fstream>
#include <sstream>
#include "StringSplitter.h"
#include <vector>
#include <iostream>
using namespace std;

void get_links(string source);

int main()
{
	StringSplitter strspl;
	vector<string> temp_vect;
	string temp_line;

	fstream source_lines;

	get_links("http://en.wikipedia.org/wiki/Lord_Voldemort");

	source_lines.open("Test.txt");

	fstream links;
	links.open("links.txt");
	links.clear();

	do{
		getline(source_lines, temp_line);
		temp_vect = strspl.split(temp_line, "\"");
		if ((temp_vect.size() > 2))
		{
			links << temp_vect[1] << endl;
		}
	} while (source_lines.good());

	source_lines.close();
	links.close();

	fstream temp_file;
	temp_file.open("temp.txt");
	links.open("links.txt");

	temp_file.clear();

	do{
		getline(links, temp_line);

		if ((temp_line != "/wiki/Help:IPA_for_English#Key") && (temp_line != "/wiki/Help:IPA_for_English") &&
			(temp_line != "/wiki/Wikipedia:Protection_policy#semi") && (temp_line[0] == '/') && (temp_line != "/wiki/Wikipedia:Citing_sources")
			&& (temp_line != "/wiki/Main_Page") && ((temp_line.size() > 7) && (temp_line[temp_line.size() - 4] != '.') &&
			(temp_line[temp_line.size() - 3] != 'j') && (temp_line[temp_line.size() - 2] != 'p') && (temp_line[temp_line.size()-1] != 'g')))

		{
			{
				temp_file << temp_line << endl;
			}
		}
	} while (links.good());

	links.close();
	temp_file.close();

	temp_file.open("temp.txt");
	links.open("links.txt");

	links.clear();
	
	do{
		getline(temp_file, temp_line);
			
		links << temp_line << endl;
	} while (temp_file.good());
	

	return 0;
}

void get_links(string source){
	fstream output;

	output.open("Test.txt");
	output.clear();

	HINTERNET connect = InternetOpen("MyBrowser", INTERNET_OPEN_TYPE_PRECONFIG, NULL, NULL, 0);

	if (!connect){
		cout << "Connection Failed or Syntax error";
	}

	HINTERNET OpenAddress = InternetOpenUrl(connect, source.c_str(), NULL, 0, INTERNET_FLAG_PRAGMA_NOCACHE | INTERNET_FLAG_KEEP_CONNECTION, 0);

	if (!OpenAddress)
	{
		DWORD ErrorNum = GetLastError();
		cout << "Failed to open URL \nError No: " << ErrorNum;
		InternetCloseHandle(connect);
	}

	string check_for_stuff;
	char DataReceived[131072];
	DWORD NumberOfBytesRead = 0;
	while (InternetReadFile(OpenAddress, DataReceived, 131072, &NumberOfBytesRead) && NumberOfBytesRead)
	{
		//if loop
		//href="/wiki/
		string data;
		data = DataReceived;
		if (data.size() > 30)
		{
			for (int i = 0; i < (data.size() - 12); i++)
			{
				if ((data[i] == 'h') && (data[i + 1] == 'r') && (data[i + 2] == 'e') && (data[i + 3] == 'f') && (data[i + 4] == '=')
					&& (data[i + 5] == '\"') && (data[i + 6] == '/') && (data[i + 7] == 'w') && (data[i + 8] == 'i') && (data[i + 9] == 'k')
					&& (data[i + 10] == 'i') && (data[i + 11] == '/'))
				{
					stringstream next_one;
					for (int j = i; j < (i + 128); j++)
					{
						next_one << data[j];
					}
					string d;
					d = next_one.str();
					const char * c = d.c_str();
					output << c << endl;
				}
			}
		} 
		//output << DataReceived;
		//cout << DataReceived;
	}

	InternetCloseHandle(OpenAddress);
	InternetCloseHandle(connect);

	output.close();
}
