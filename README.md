# Real_Chathook
I created a new repo to preserve timestamps of the repository: https://github.com/andrewpisula/Chathook

```cpp
// This code was written week of December 20th, 2018.
// I suppose with Roblox's introduction to Byfron making this code obsolete, I can stop gatekeeping this.
// This uses Sloppey's INT3 Locator function. This is fragments of the Eros Source code ripped out and placed into a single file.

#pragma once

typedef DWORD(__cdecl* alua_pushcclosure)(DWORD a1, int a2, int a3, int a4, int a5);// _DWORD *a1, int a2, int a3, int a4, int a5
alua_pushcclosure raw_lua_pushcclosure = (alua_pushcclosure)unprotect(os(0x7D6F90)); // sub_7CBC40(a1, a2[1], 0, 0, 0);


namespace CallCheck {
	std::vector<DWORD> BreakpointList;
	std::vector<DWORD> FunctionList;

	LONG WINAPI VectoredExceptionHandler(PEXCEPTION_POINTERS exception) {
		if (exception->ExceptionRecord->ExceptionCode == EXCEPTION_BREAKPOINT) {
			for (int i = 0; i < FunctionList.size(); ++i) {
				if (exception->ContextRecord->Eip == BreakpointList[i]) {
					//std::cout << "Breakpoint[" << i << "] recognized." << std::endl;//debugging purposes
					exception->ContextRecord->Eip = (DWORD)FunctionList[i];
					//std::cout << "Function[" << i << "] set." << std::endl;//debugging purposes
					//std::cout << "return EXCEPTION_CONTINUE_EXECUTION;" << std::endl;//debugging purposes
					return EXCEPTION_CONTINUE_EXECUTION;
				}
			}
		}
		return EXCEPTION_CONTINUE_SEARCH;
	}

	DWORD locateINT3() {
		DWORD _s = os(0x400000);
		const char i3_8opcode[8] = { 0xCC, 0xCC, 0xCC, 0xCC, 0xCC, 0xCC, 0xCC, 0xCC };
		for (int i = 0; i < INT_MAX; i++) {
			if (memcmp((void*)(_s + i), i3_8opcode, sizeof(i3_8opcode)) == 0) {
				return (_s + i);
			}
		}
		return NULL;
	}

	void InitializeCallCheck() {
		AddVectoredExceptionHandler(1, &VectoredExceptionHandler);
	}

	void fixed_pushcclosure(DWORD a1, DWORD a2, int a3, int a4, int a5) {
		BreakpointList.push_back(locateINT3());
		FunctionList.push_back(a2);
		raw_lua_pushcclosure(a1, BreakpointList[BreakpointList.size() - 1], a3, a4, a5);
	}
}

#define lua_pushcclosure(State, Function, n) CallCheck::fixed_pushcclosure(State, (DWORD)Function, n, 0, 0)

#define lua_pushcfunction(L,f)	lua_pushcclosure(L, (f), 0)

#define lua_register(L,n,f) lua_pushcfunction(L, f); \
							lua_setglobal(L, n);

int ChatHook_Handler(DWORD state) {
	if (ChatHookOn) {
		const char* Input = lua_tolstring(state, -2, 0);
		if (std::string(Input).substr(0, 1) == ChatHookPrefix) {
			ChatHook_RunCommand(std::string(Input).substr(1, std::string::npos));
		}
	}
	return 0;
}

void InitializeChatHook() {
	lua_getglobal(ls, "game");
	lua_getfield(ls, -1, "Players");
	lua_getfield(ls, -1, "LocalPlayer");
	lua_getfield(ls, -1, "Chatted");
	lua_getfield(ls, -1, "connect");
	lua_pushvalue(ls, -2);
	lua_pushcfunction(ls, ChatHook_Handler);
	lua_pcall(ls, 2, 0, 0);
	lua_emptystack(ls);
}
```
