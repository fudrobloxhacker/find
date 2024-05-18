# find
call funcs with the address of this pattern inside jvm module
```cpp
     auto sdk::get_class_list(void* handle, unsigned __int64 address) -> std::vector<unsigned __int64>
    {
    	std::vector<unsigned __int64> classes;
     
    	unsigned __int64 dict = 0;
     
    	ReadProcessMemory(handle, (void*)(address + 0x5a), &dict, 0x8, nullptr);
     
    	unsigned __int32 size = 0;
     
    	ReadProcessMemory(handle, (void*)dict, &size, 0x4, nullptr);
     
    	if (size != 0)
    	{
    		unsigned __int64 base = 0;
     
    		ReadProcessMemory(handle, (void*)(dict + 0x8), &base, 0x8, nullptr);
     
    		for (unsigned __int32 x = 0, s = size; x < s; ++x)
    		{
    			unsigned __int64 entry = 0;
     
    			ReadProcessMemory(handle, (void*)(base + x * 0x8), &entry, 0x8, nullptr);
     
    			unsigned __int64 class_entry = 0;
     
    			ReadProcessMemory(handle, (void*)(entry + 0x8), &class_entry, 0x8, nullptr);
     
    			for (; class_entry != 0;)
    			{
    				unsigned __int64 mirror = 0;
     
    				ReadProcessMemory(handle, (void*)(class_entry + 0x10), &mirror, 0x8, nullptr);
     
    				if (mirror != 0) classes.emplace_back(mirror);
     
    				ReadProcessMemory(handle, (void*)(class_entry + 0x8), &class_entry, 0x8, nullptr);
    			}
    		}
    	}
     
    	return classes;
    }
     
    auto sdk::find_class(void* handle, unsigned __int64 address, std::string name) -> unsigned __int64
    {
    	unsigned __int64 ret_mirror = 0;
     
    	for (const auto& mirror : sdk::get_class_list(handle, address))
    	{
    		unsigned __int64 symbol = 0;
     
    		ReadProcessMemory(handle, (void*)(mirror + 0x10), &symbol, 0x8, nullptr);
     
    		if (symbol != 0)
    		{
    			unsigned short size = 0;
     
    			ReadProcessMemory(handle, (void*)symbol, &size, 0x2, nullptr);
     
    			std::string buf(size, 0);
     
    			ReadProcessMemory(handle, (void*)(symbol + 0x8), &buf[0], size, nullptr);
     
    			if (buf == name) ret_mirror = mirror;
    		}
    	}
     
    	return ret_mirror;
    }
