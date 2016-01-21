---
layout: page
title:  Определение загрузки процессора в Linux на C++
date:   2011-06-16 00:00:00
---

Довольно много времени я потратил на то, чтобы найти, как правильно реализовать на плюсах определение загрузки процессора под Linux.
Первый вариант использовал popen для чтения вывода top, но по непонятным причинам с каждой итерацией работы такой программы появлялось
все больше зомби-процессов sh – вероятно, помимо `pclose` надо было еще что-то делать после чтения вывода порождаемого процесса.
Второй вариант решения задачи оказался для меня более приемлемым, его и приведу тут.

<!--break-->

{% highlight cpp %}
#include <vector>
#include <cstdlib>
#include <iostream>
#include <fstream>

std::vector<float> readCpuStats()
{
	std::vector<float> ret;
	std::ifstream stat_file("/proc/stat");
	if (!stat_file.is_open())
	{
		std::cout << "Unable to open /proc/stat" << std::endl;
		return ret;
	}

	int val;
	std::string tmp;
	stat_file >> tmp;
	for (int i = 0; i < 4; ++i)
	{
		stat_file >> val;
		ret.push_back(val);
	}

	stat_file.close();

	return ret;
}


int getCpuLoad(int dt)
{
	std::vector<float> stats1 = readCpuStats();
	sleep(dt);
	std::vector<float> stats2 = readCpuStats();

	int size1 = stats1.size();
	int size2 = stats2.size();

	if (!size1 || !size2 || size1 != size2)
		return 2;

	for (int i = 0; i < size1; ++i)
		stats2[i] -= stats1[i];

	int sum = 1;
	for (int i = 0; i < size1; ++i)
		sum += stats2[i];
	int load = 100 - (stats2[size2 - 1] * 100 / sum);

	return load;
}


int main(int argc, char *argv[])
{
	if (argc == 1)
	{
		std::cout << "Usage: " << argv[0] << " <sample_interval>"
				<< std::endl;
		return 1;
	}

	int interval = atoi(argv[1]);
	int load = getCpuLoad(interval);
	std::cout << load << std::endl;
	return 0;
}
{% endhighlight %}