// counts each instance a button is pressed in KSH files

#include <iostream>
#include <fstream>
#include <string>
#include <sstream>
#include <filesystem>

using namespace std;
namespace fs = std::filesystem;

void countKSH(string filename, int ButtonTotal[], int HighestCount[], string HighestChart[], double HighestSkewed[], string HighestCSkewed[]) {
	int Button[7] = { 0, 0, 0, 0, 0, 0, 0 };
	bool hold[7] = { 0, 0, 0, 0, 0, 0, 0 };
	bool highDiff = 0;
	
	//Open KSH
	ifstream inFile;
	string line;
	inFile.open(filename);
	if (!inFile.is_open()) {
		cout << "Failed to open file: " << filename << endl;
	}

	//Read KSH line by line and add to BT/FX array if it's found
	while (getline(inFile, line)) {
		if (line == "level=17" || line == "level=18" || line == "level=19" || line == "level=20") { //set flag so that it only counts if level is 17+
			highDiff = 1;
		}
		if (line.length() > 9) { //avoid char doesnt exist error
			if ((line.at(0) == '0' || line.at(0) == '1') && (line.at(1) == '0' || line.at(1) == '1') && (highDiff == 1)) {
				//Check for BT
				for (int i = 0; i < 4; i++) {
					//Check for no button to reset hold
					if (line.at(i) == '0') {
						hold[i] = 0;
					}
					//Check for button and reset hold
					else if (line.at(i) == '1') {
						Button[i]++;
						hold[i] = 0;
					}
					//Check for hold
					else if (line.at(i) == '2' && hold[i] == 0) {
						Button[i]++;
						hold[i] = 1;
					}
				}

				//Check for FX chip
				for (int i = 5; i < 7; i++) {
					//Check for no fx to reset hold
					if (line.at(i) == '0') {
						hold[i] = 0;
					}
					//Check for fx chip and reset hold
					else if (line.at(i) == '2') {
						Button[i]++;
						hold[i] = 0;
					}
					//Check for fx hold
					else if (line.at(i) == '1' && hold[i] == 0) {
						Button[i]++;
						hold[i] = 1;
					}
				}
			}
		}
	}
	inFile.close();

	//add to total
	for (int i = 0; i < 7; i++) {
		ButtonTotal[i] = ButtonTotal[i] + Button[i];
	}

	//Check if button count is abnormally high
	for (int i = 0; i < 7; i++) {
		if (Button[i] > HighestCount[i]) {
			HighestCount[i] = Button[i];
			HighestChart[i] = filename;
		}
	}

	//Check if fx combined count is abnormally high
	if (Button[5] + Button[6] > HighestCount[4]) {
		HighestCount[4] = Button[5] + Button[6];
		HighestChart[4] = filename;
	}

	//Check which charts are skewed most towards each button
	double Btotal = 0.0;
	for (int i = 0; i < 7; i++) {
		Btotal = Btotal + Button[i];
	}
	if (Btotal != 0) {
		for (int i = 0; i < 7; i++) {
			double BPercent = (Button[i] / Btotal) * 100;
			if (BPercent > HighestSkewed[i]) {
				HighestSkewed[i] = BPercent;
				HighestCSkewed[i] = filename;
			}
		}
	}

	if (Button[0] != 0) {
		cout << "BT_A: " << Button[0] << endl;
		cout << "BT_B: " << Button[1] << endl;
		cout << "BT_C: " << Button[2] << endl;
		cout << "BT_D: " << Button[3] << endl;
		cout << "FX_L: " << Button[5] << endl;
		cout << "FX_R: " << Button[6] << endl;
	}
	return;
}

int main()
{
	int ButtonTotal[7] = { 0, 0, 0, 0, 0, 0, 0 };
	int HighestCount[7] = { 0, 0, 0, 0, 0, 0, 0 };
	double HighestSkewed[7] = { 0, 0, 0, 0, 0, 0, 0 };
	string filename;
	string HighestChart[7];
	string HighestCSkewed[7];

	//Ask for KSH file name
	cout << "Input KSH song folder directory to check: ";
	getline(cin, filename);
	cout << endl;

	//Search directory for .ksh
	for (const auto& entry : fs::recursive_directory_iterator(filename)) {
		const auto filenameStr = entry.path().filename().string();
		if (entry.is_directory()) {
			std::cout << "\ndir:  " << filenameStr << '\n';
		}
		else if (entry.is_regular_file()) {
			auto filenameType = entry.path().extension().string();
			//std::cout << "file: " << filenameStr << "   type: " << filenameType << '\n';
			if (filenameType == ".ksh") { //if .ksh is found, count buttons
				cout << entry.path().string() << endl;
				countKSH(entry.path().string(), ButtonTotal, HighestCount, HighestChart, HighestSkewed, HighestCSkewed);
			}
		}
		else
			std::cout << "??    " << filenameStr << '\n';
	}

	cout << endl;
	cout << "Most BT_A used out of any chart: " << HighestCount[0] << " on chart: " << HighestChart[0] << endl;
	cout << "Most BT_B used out of any chart: " << HighestCount[1] << " on chart: " << HighestChart[1] << endl;
	cout << "Most BT_C used out of any chart: " << HighestCount[2] << " on chart: " << HighestChart[2] << endl;
	cout << "Most BT_D used out of any chart: " << HighestCount[3] << " on chart: " << HighestChart[3] << endl;
	cout << "Most FX_L used out of any chart: " << HighestCount[5] << " on chart: " << HighestChart[5] << endl;
	cout << "Most FX_R used out of any chart: " << HighestCount[6] << " on chart: " << HighestChart[6] << endl;
	cout << "Most FX_combined used out of any chart: " << HighestCount[4] << " on chart: " << HighestChart[4] << endl;
	
	cout << endl << "Charts most skewed towards a single button:" << endl;
	cout << HighestSkewed[0] << "\% Skewed towards BT_A. Chart: " << HighestCSkewed[0] << endl;
	cout << HighestSkewed[1] << "\% Skewed towards BT_B. Chart: " << HighestCSkewed[1] << endl;
	cout << HighestSkewed[2] << "\% Skewed towards BT_C. Chart: " << HighestCSkewed[2] << endl;
	cout << HighestSkewed[3] << "\% Skewed towards BT_D. Chart: " << HighestCSkewed[3] << endl;
	cout << HighestSkewed[5] << "\% Skewed towards FX_L. Chart: " << HighestCSkewed[5] << endl;
	cout << HighestSkewed[6] << "\% Skewed towards FX_R. Chart: " << HighestCSkewed[6] << endl;

	cout << endl;
	cout << "BT_A Total: " << ButtonTotal[0] << endl;
	cout << "BT_B Total: " << ButtonTotal[1] << endl;
	cout << "BT_C Total: " << ButtonTotal[2] << endl;
	cout << "BT_D Total: " << ButtonTotal[3] << endl;
	cout << "FX_L Total: " << ButtonTotal[5] << endl;
	cout << "FX_R Total: " << ButtonTotal[6] << endl;

	system("pause");

	return 0;
}

