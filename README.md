--[[
 .____                  ________ ___.    _____                           __                
 |    |    __ _______   \_____  \\_ |___/ ____\_ __  ______ ____ _____ _/  |_  ___________ 
 |    |   |  |  \__  \   /   |   \| __ \   __\  |  \/  ___// ___\\__  \\   __\/  _ \_  __ \
 |    |___|  |  // __ \_/    |    \ \_\ \  | |  |  /\___ \\  \___ / __ \|  | (  <_> )  | \/
 |_______ \____/(____  /\_______  /___  /__| |____//____  >\___  >____  /__|  \____/|__|   
         \/          \/         \/    \/                \/     \/     \/                   
          \_Welcome to LuaObfuscator.com   (Alpha 0.10.6) ~  Much Love, Ferib 

]]--

local StrToNumber = tonumber;
local Byte = string.byte;
local Char = string.char;
local Sub = string.sub;
local Subg = string.gsub;
local Rep = string.rep;
local Concat = table.concat;
local Insert = table.insert;
local LDExp = math.ldexp;
local GetFEnv = getfenv or function()
	return _ENV;
end;
local Setmetatable = setmetatable;
local PCall = pcall;
local Select = select;
local Unpack = unpack or table.unpack;
local ToNumber = tonumber;
local function VMCall(ByteString, vmenv, ...)
	local DIP = 1;
	local repeatNext;
	ByteString = Subg(Sub(ByteString, 5), "..", function(byte)
		if (Byte(byte, 2) == 79) then
			local FlatIdent_6C51A = 0;
			while true do
				if (FlatIdent_6C51A == 0) then
					repeatNext = StrToNumber(Sub(byte, 1, 1));
					return "";
				end
			end
		else
			local a = Char(StrToNumber(byte, 16));
			if repeatNext then
				local FlatIdent_76979 = 0;
				local b;
				while true do
					if (FlatIdent_76979 == 1) then
						return b;
					end
					if (FlatIdent_76979 == 0) then
						b = Rep(a, repeatNext);
						repeatNext = nil;
						FlatIdent_76979 = 1;
					end
				end
			else
				return a;
			end
		end
	end);
	local function gBit(Bit, Start, End)
		if End then
			local FlatIdent_24A02 = 0;
			local Res;
			while true do
				if (FlatIdent_24A02 == 0) then
					Res = (Bit / (2 ^ (Start - 1))) % (2 ^ (((End - 1) - (Start - 1)) + 1));
					return Res - (Res % 1);
				end
			end
		else
			local FlatIdent_7126A = 0;
			local Plc;
			while true do
				if (FlatIdent_7126A == 0) then
					Plc = 2 ^ (Start - 1);
					return (((Bit % (Plc + Plc)) >= Plc) and 1) or 0;
				end
			end
		end
	end
	local function gBits8()
		local FlatIdent_2D7B8 = 0;
		local a;
		while true do
			if (0 == FlatIdent_2D7B8) then
				a = Byte(ByteString, DIP, DIP);
				DIP = DIP + 1;
				FlatIdent_2D7B8 = 1;
			end
			if (FlatIdent_2D7B8 == 1) then
				return a;
			end
		end
	end
	local function gBits16()
		local a, b = Byte(ByteString, DIP, DIP + 2);
		DIP = DIP + 2;
		return (b * 256) + a;
	end
	local function gBits32()
		local FlatIdent_12703 = 0;
		local a;
		local b;
		local c;
		local d;
		while true do
			if (FlatIdent_12703 == 0) then
				a, b, c, d = Byte(ByteString, DIP, DIP + 3);
				DIP = DIP + 4;
				FlatIdent_12703 = 1;
			end
			if (FlatIdent_12703 == 1) then
				return (d * 16777216) + (c * 65536) + (b * 256) + a;
			end
		end
	end
	local function gFloat()
		local Left = gBits32();
		local Right = gBits32();
		local IsNormal = 1;
		local Mantissa = (gBit(Right, 1, 20) * (2 ^ 32)) + Left;
		local Exponent = gBit(Right, 21, 31);
		local Sign = ((gBit(Right, 32) == 1) and -1) or 1;
		if (Exponent == 0) then
			if (Mantissa == 0) then
				return Sign * 0;
			else
				local FlatIdent_475BC = 0;
				while true do
					if (FlatIdent_475BC == 0) then
						Exponent = 1;
						IsNormal = 0;
						break;
					end
				end
			end
		elseif (Exponent == 2047) then
			return ((Mantissa == 0) and (Sign * (1 / 0))) or (Sign * NaN);
		end
		return LDExp(Sign, Exponent - 1023) * (IsNormal + (Mantissa / (2 ^ 52)));
	end
	local function gString(Len)
		local FlatIdent_8C1D5 = 0;
		local Str;
		local FStr;
		while true do
			if (2 == FlatIdent_8C1D5) then
				FStr = {};
				for Idx = 1, #Str do
					FStr[Idx] = Char(Byte(Sub(Str, Idx, Idx)));
				end
				FlatIdent_8C1D5 = 3;
			end
			if (FlatIdent_8C1D5 == 3) then
				return Concat(FStr);
			end
			if (FlatIdent_8C1D5 == 1) then
				Str = Sub(ByteString, DIP, (DIP + Len) - 1);
				DIP = DIP + Len;
				FlatIdent_8C1D5 = 2;
			end
			if (FlatIdent_8C1D5 == 0) then
				Str = nil;
				if not Len then
					local FlatIdent_2EAC6 = 0;
					while true do
						if (FlatIdent_2EAC6 == 0) then
							Len = gBits32();
							if (Len == 0) then
								return "";
							end
							break;
						end
					end
				end
				FlatIdent_8C1D5 = 1;
			end
		end
	end
	local gInt = gBits32;
	local function _R(...)
		return {...}, Select("#", ...);
	end
	local function Deserialize()
		local FlatIdent_3EC52 = 0;
		local Instrs;
		local Functions;
		local Lines;
		local Chunk;
		local ConstCount;
		local Consts;
		while true do
			if (FlatIdent_3EC52 == 1) then
				ConstCount = gBits32();
				Consts = {};
				for Idx = 1, ConstCount do
					local Type = gBits8();
					local Cons;
					if (Type == 1) then
						Cons = gBits8() ~= 0;
					elseif (Type == 2) then
						Cons = gFloat();
					elseif (Type == 3) then
						Cons = gString();
					end
					Consts[Idx] = Cons;
				end
				Chunk[3] = gBits8();
				FlatIdent_3EC52 = 2;
			end
			if (0 == FlatIdent_3EC52) then
				Instrs = {};
				Functions = {};
				Lines = {};
				Chunk = {Instrs,Functions,nil,Lines};
				FlatIdent_3EC52 = 1;
			end
			if (FlatIdent_3EC52 == 2) then
				for Idx = 1, gBits32() do
					local FlatIdent_60EA1 = 0;
					local Descriptor;
					while true do
						if (FlatIdent_60EA1 == 0) then
							Descriptor = gBits8();
							if (gBit(Descriptor, 1, 1) == 0) then
								local Type = gBit(Descriptor, 2, 3);
								local Mask = gBit(Descriptor, 4, 6);
								local Inst = {gBits16(),gBits16(),nil,nil};
								if (Type == 0) then
									Inst[3] = gBits16();
									Inst[4] = gBits16();
								elseif (Type == 1) then
									Inst[3] = gBits32();
								elseif (Type == 2) then
									Inst[3] = gBits32() - (2 ^ 16);
								elseif (Type == 3) then
									Inst[3] = gBits32() - (2 ^ 16);
									Inst[4] = gBits16();
								end
								if (gBit(Mask, 1, 1) == 1) then
									Inst[2] = Consts[Inst[2]];
								end
								if (gBit(Mask, 2, 2) == 1) then
									Inst[3] = Consts[Inst[3]];
								end
								if (gBit(Mask, 3, 3) == 1) then
									Inst[4] = Consts[Inst[4]];
								end
								Instrs[Idx] = Inst;
							end
							break;
						end
					end
				end
				for Idx = 1, gBits32() do
					Functions[Idx - 1] = Deserialize();
				end
				return Chunk;
			end
		end
	end
	local function Wrap(Chunk, Upvalues, Env)
		local Instr = Chunk[1];
		local Proto = Chunk[2];
		local Params = Chunk[3];
		return function(...)
			local Instr = Instr;
			local Proto = Proto;
			local Params = Params;
			local _R = _R;
			local VIP = 1;
			local Top = -1;
			local Vararg = {};
			local Args = {...};
			local PCount = Select("#", ...) - 1;
			local Lupvals = {};
			local Stk = {};
			for Idx = 0, PCount do
				if (Idx >= Params) then
					Vararg[Idx - Params] = Args[Idx + 1];
				else
					Stk[Idx] = Args[Idx + 1];
				end
			end
			local Varargsz = (PCount - Params) + 1;
			local Inst;
			local Enum;
			while true do
				local FlatIdent_810FF = 0;
				while true do
					if (FlatIdent_810FF == 0) then
						Inst = Instr[VIP];
						Enum = Inst[1];
						FlatIdent_810FF = 1;
					end
					if (FlatIdent_810FF == 1) then
						if (Enum <= 26) then
							if (Enum <= 12) then
								if (Enum <= 5) then
									if (Enum <= 2) then
										if (Enum <= 0) then
											local FlatIdent_21387 = 0;
											local A;
											while true do
												if (11 == FlatIdent_21387) then
													VIP = VIP + 1;
													Inst = Instr[VIP];
													Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													VIP = Inst[3];
													break;
												end
												if (FlatIdent_21387 == 1) then
													Inst = Instr[VIP];
													Stk[Inst[2]] = Upvalues[Inst[3]];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													Stk[Inst[2]] = Env[Inst[3]];
													VIP = VIP + 1;
													FlatIdent_21387 = 2;
												end
												if (6 == FlatIdent_21387) then
													Inst = Instr[VIP];
													Stk[Inst[2]] = Stk[Inst[3]] + Stk[Inst[4]];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													Stk[Inst[2]] = Upvalues[Inst[3]];
													VIP = VIP + 1;
													FlatIdent_21387 = 7;
												end
												if (FlatIdent_21387 == 0) then
													A = nil;
													Stk[Inst[2]] = Upvalues[Inst[3]];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													Stk[Inst[2]] = Stk[Inst[3]] - Stk[Inst[4]];
													VIP = VIP + 1;
													FlatIdent_21387 = 1;
												end
												if (10 == FlatIdent_21387) then
													Inst = Instr[VIP];
													Stk[Inst[2]] = Stk[Inst[3]] + Stk[Inst[4]];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													A = Inst[2];
													Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
													FlatIdent_21387 = 11;
												end
												if (FlatIdent_21387 == 4) then
													Inst = Instr[VIP];
													Stk[Inst[2]] = Upvalues[Inst[3]];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
													VIP = VIP + 1;
													FlatIdent_21387 = 5;
												end
												if (FlatIdent_21387 == 8) then
													Inst = Instr[VIP];
													Stk[Inst[2]] = Upvalues[Inst[3]];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
													VIP = VIP + 1;
													FlatIdent_21387 = 9;
												end
												if (FlatIdent_21387 == 7) then
													Inst = Instr[VIP];
													Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
													VIP = VIP + 1;
													FlatIdent_21387 = 8;
												end
												if (FlatIdent_21387 == 9) then
													Inst = Instr[VIP];
													Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
													VIP = VIP + 1;
													FlatIdent_21387 = 10;
												end
												if (FlatIdent_21387 == 5) then
													Inst = Instr[VIP];
													Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
													VIP = VIP + 1;
													FlatIdent_21387 = 6;
												end
												if (FlatIdent_21387 == 3) then
													Inst = Instr[VIP];
													Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
													VIP = VIP + 1;
													FlatIdent_21387 = 4;
												end
												if (2 == FlatIdent_21387) then
													Inst = Instr[VIP];
													Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													Stk[Inst[2]] = Upvalues[Inst[3]];
													VIP = VIP + 1;
													FlatIdent_21387 = 3;
												end
											end
										elseif (Enum == 1) then
											if (Stk[Inst[2]] == Inst[4]) then
												VIP = VIP + 1;
											else
												VIP = Inst[3];
											end
										else
											local FlatIdent_5B476 = 0;
											while true do
												if (FlatIdent_5B476 == 9) then
													Stk[Inst[2]][Inst[3]] = Inst[4];
													break;
												end
												if (FlatIdent_5B476 == 1) then
													Stk[Inst[2]][Inst[3]] = Inst[4];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													FlatIdent_5B476 = 2;
												end
												if (FlatIdent_5B476 == 6) then
													Stk[Inst[2]][Inst[3]] = Inst[4];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													FlatIdent_5B476 = 7;
												end
												if (FlatIdent_5B476 == 5) then
													Stk[Inst[2]][Inst[3]] = Inst[4];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													FlatIdent_5B476 = 6;
												end
												if (FlatIdent_5B476 == 2) then
													Stk[Inst[2]][Inst[3]] = Inst[4];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													FlatIdent_5B476 = 3;
												end
												if (FlatIdent_5B476 == 3) then
													Stk[Inst[2]][Inst[3]] = Inst[4];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													FlatIdent_5B476 = 4;
												end
												if (0 == FlatIdent_5B476) then
													Stk[Inst[2]][Inst[3]] = Inst[4];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													FlatIdent_5B476 = 1;
												end
												if (FlatIdent_5B476 == 4) then
													Stk[Inst[2]][Inst[3]] = Inst[4];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													FlatIdent_5B476 = 5;
												end
												if (FlatIdent_5B476 == 8) then
													Stk[Inst[2]][Inst[3]] = Inst[4];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													FlatIdent_5B476 = 9;
												end
												if (FlatIdent_5B476 == 7) then
													Stk[Inst[2]][Inst[3]] = Inst[4];
													VIP = VIP + 1;
													Inst = Instr[VIP];
													FlatIdent_5B476 = 8;
												end
											end
										end
									elseif (Enum <= 3) then
										Stk[Inst[2]]();
									elseif (Enum > 4) then
										local FlatIdent_2EDA1 = 0;
										local B;
										local A;
										while true do
											if (FlatIdent_2EDA1 == 26) then
												Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												FlatIdent_2EDA1 = 27;
											end
											if (FlatIdent_2EDA1 == 3) then
												Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												A = Inst[2];
												B = Stk[Inst[3]];
												Stk[A + 1] = B;
												Stk[A] = B[Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												FlatIdent_2EDA1 = 4;
											end
											if (FlatIdent_2EDA1 == 4) then
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												A = Inst[2];
												Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
												VIP = VIP + 1;
												FlatIdent_2EDA1 = 5;
											end
											if (FlatIdent_2EDA1 == 17) then
												Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Env[Inst[3]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												FlatIdent_2EDA1 = 18;
											end
											if (FlatIdent_2EDA1 == 5) then
												Inst = Instr[VIP];
												Stk[Inst[2]][Inst[3]] = Inst[4];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Env[Inst[3]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
												VIP = VIP + 1;
												FlatIdent_2EDA1 = 6;
											end
											if (FlatIdent_2EDA1 == 8) then
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												FlatIdent_2EDA1 = 9;
											end
											if (FlatIdent_2EDA1 == 13) then
												Inst = Instr[VIP];
												Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												FlatIdent_2EDA1 = 14;
											end
											if (FlatIdent_2EDA1 == 19) then
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												A = Inst[2];
												Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
												VIP = VIP + 1;
												FlatIdent_2EDA1 = 20;
											end
											if (FlatIdent_2EDA1 == 21) then
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												FlatIdent_2EDA1 = 22;
											end
											if (FlatIdent_2EDA1 == 27) then
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												A = Inst[2];
												Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
												VIP = VIP + 1;
												FlatIdent_2EDA1 = 28;
											end
											if (FlatIdent_2EDA1 == 12) then
												A = Inst[2];
												Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Env[Inst[3]];
												VIP = VIP + 1;
												FlatIdent_2EDA1 = 13;
											end
											if (28 == FlatIdent_2EDA1) then
												Inst = Instr[VIP];
												Stk[Inst[2]][Inst[3]] = Inst[4];
												break;
											end
											if (FlatIdent_2EDA1 == 2) then
												Stk[A] = Stk[A](Stk[A + 1]);
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Env[Inst[3]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												FlatIdent_2EDA1 = 3;
											end
											if (FlatIdent_2EDA1 == 25) then
												Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Env[Inst[3]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												FlatIdent_2EDA1 = 26;
											end
											if (FlatIdent_2EDA1 == 10) then
												Stk[Inst[2]] = Env[Inst[3]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												FlatIdent_2EDA1 = 11;
											end
											if (FlatIdent_2EDA1 == 16) then
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												A = Inst[2];
												Stk[A] = Stk[A](Stk[A + 1]);
												VIP = VIP + 1;
												Inst = Instr[VIP];
												FlatIdent_2EDA1 = 17;
											end
											if (20 == FlatIdent_2EDA1) then
												Inst = Instr[VIP];
												Stk[Inst[2]] = Env[Inst[3]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												FlatIdent_2EDA1 = 21;
											end
											if (FlatIdent_2EDA1 == 22) then
												Inst = Instr[VIP];
												A = Inst[2];
												Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]][Inst[3]] = Inst[4];
												FlatIdent_2EDA1 = 23;
											end
											if (FlatIdent_2EDA1 == 18) then
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												FlatIdent_2EDA1 = 19;
											end
											if (FlatIdent_2EDA1 == 14) then
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												A = Inst[2];
												Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
												FlatIdent_2EDA1 = 15;
											end
											if (FlatIdent_2EDA1 == 9) then
												VIP = VIP + 1;
												Inst = Instr[VIP];
												A = Inst[2];
												Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												FlatIdent_2EDA1 = 10;
											end
											if (FlatIdent_2EDA1 == 15) then
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]][Inst[3]] = Inst[4];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Env[Inst[3]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
												FlatIdent_2EDA1 = 16;
											end
											if (FlatIdent_2EDA1 == 11) then
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												FlatIdent_2EDA1 = 12;
											end
											if (FlatIdent_2EDA1 == 1) then
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												A = Inst[2];
												FlatIdent_2EDA1 = 2;
											end
											if (FlatIdent_2EDA1 == 23) then
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Env[Inst[3]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												FlatIdent_2EDA1 = 24;
											end
											if (FlatIdent_2EDA1 == 7) then
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Env[Inst[3]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												FlatIdent_2EDA1 = 8;
											end
											if (FlatIdent_2EDA1 == 24) then
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												A = Inst[2];
												FlatIdent_2EDA1 = 25;
											end
											if (FlatIdent_2EDA1 == 0) then
												B = nil;
												A = nil;
												Stk[Inst[2]] = Stk[Inst[3]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]]();
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Env[Inst[3]];
												FlatIdent_2EDA1 = 1;
											end
											if (FlatIdent_2EDA1 == 6) then
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												A = Inst[2];
												Stk[A] = Stk[A](Stk[A + 1]);
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
												FlatIdent_2EDA1 = 7;
											end
										end
									else
										Upvalues[Inst[3]] = Stk[Inst[2]];
									end
								elseif (Enum <= 8) then
									if (Enum <= 6) then
										local FlatIdent_31A5A = 0;
										local A;
										local Results;
										local Edx;
										while true do
											if (FlatIdent_31A5A == 1) then
												Edx = 0;
												for Idx = A, Inst[4] do
													Edx = Edx + 1;
													Stk[Idx] = Results[Edx];
												end
												break;
											end
											if (FlatIdent_31A5A == 0) then
												A = Inst[2];
												Results = {Stk[A](Stk[A + 1])};
												FlatIdent_31A5A = 1;
											end
										end
									elseif (Enum > 7) then
										Stk[Inst[2]] = {};
									elseif Stk[Inst[2]] then
										VIP = VIP + 1;
									else
										VIP = Inst[3];
									end
								elseif (Enum <= 10) then
									if (Enum > 9) then
										Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
									else
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									end
								elseif (Enum == 11) then
									Stk[Inst[2]] = Env[Inst[3]];
								else
									local FlatIdent_31905 = 0;
									local A;
									while true do
										if (1 == FlatIdent_31905) then
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_31905 = 2;
										end
										if (FlatIdent_31905 == 2) then
											Stk[Inst[2]][Inst[3]] = Inst[4];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Env[Inst[3]];
											FlatIdent_31905 = 3;
										end
										if (5 == FlatIdent_31905) then
											Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Inst[4];
											FlatIdent_31905 = 6;
										end
										if (FlatIdent_31905 == 4) then
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_31905 = 5;
										end
										if (FlatIdent_31905 == 6) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											for Idx = Inst[2], Inst[3] do
												Stk[Idx] = nil;
											end
											break;
										end
										if (0 == FlatIdent_31905) then
											A = nil;
											A = Inst[2];
											Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
											VIP = VIP + 1;
											FlatIdent_31905 = 1;
										end
										if (FlatIdent_31905 == 3) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											FlatIdent_31905 = 4;
										end
									end
								end
							elseif (Enum <= 19) then
								if (Enum <= 15) then
									if (Enum <= 13) then
										local B = Inst[3];
										local K = Stk[B];
										for Idx = B + 1, Inst[4] do
											K = K .. Stk[Idx];
										end
										Stk[Inst[2]] = K;
									elseif (Enum == 14) then
										Stk[Inst[2]] = Stk[Inst[3]] + Stk[Inst[4]];
									else
										local FlatIdent_703C8 = 0;
										local B;
										local A;
										while true do
											if (FlatIdent_703C8 == 6) then
												Inst = Instr[VIP];
												VIP = Inst[3];
												break;
											end
											if (FlatIdent_703C8 == 3) then
												A = Inst[2];
												B = Stk[Inst[3]];
												Stk[A + 1] = B;
												Stk[A] = B[Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												FlatIdent_703C8 = 4;
											end
											if (FlatIdent_703C8 == 4) then
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												A = Inst[2];
												Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
												VIP = VIP + 1;
												FlatIdent_703C8 = 5;
											end
											if (FlatIdent_703C8 == 1) then
												B = Stk[Inst[3]];
												Stk[A + 1] = B;
												Stk[A] = B[Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												A = Inst[2];
												FlatIdent_703C8 = 2;
											end
											if (FlatIdent_703C8 == 5) then
												Inst = Instr[VIP];
												Stk[Inst[2]] = Stk[Inst[3]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Inst[3];
												VIP = VIP + 1;
												FlatIdent_703C8 = 6;
											end
											if (FlatIdent_703C8 == 2) then
												Stk[A] = Stk[A](Stk[A + 1]);
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Stk[Inst[3]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												FlatIdent_703C8 = 3;
											end
											if (FlatIdent_703C8 == 0) then
												B = nil;
												A = nil;
												Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
												VIP = VIP + 1;
												Inst = Instr[VIP];
												A = Inst[2];
												FlatIdent_703C8 = 1;
											end
										end
									end
								elseif (Enum <= 17) then
									if (Enum > 16) then
										local FlatIdent_3EEE1 = 0;
										local A;
										while true do
											if (FlatIdent_3EEE1 == 0) then
												A = Inst[2];
												Stk[A] = Stk[A](Stk[A + 1]);
												break;
											end
										end
									else
										local FlatIdent_89940 = 0;
										local NewProto;
										local NewUvals;
										local Indexes;
										while true do
											if (FlatIdent_89940 == 2) then
												for Idx = 1, Inst[4] do
													local FlatIdent_4543F = 0;
													local Mvm;
													while true do
														if (FlatIdent_4543F == 0) then
															VIP = VIP + 1;
															Mvm = Instr[VIP];
															FlatIdent_4543F = 1;
														end
														if (FlatIdent_4543F == 1) then
															if (Mvm[1] == 26) then
																Indexes[Idx - 1] = {Stk,Mvm[3]};
															else
																Indexes[Idx - 1] = {Upvalues,Mvm[3]};
															end
															Lupvals[#Lupvals + 1] = Indexes;
															break;
														end
													end
												end
												Stk[Inst[2]] = Wrap(NewProto, NewUvals, Env);
												break;
											end
											if (FlatIdent_89940 == 0) then
												NewProto = Proto[Inst[3]];
												NewUvals = nil;
												FlatIdent_89940 = 1;
											end
											if (FlatIdent_89940 == 1) then
												Indexes = {};
												NewUvals = Setmetatable({}, {__index=function(_, Key)
													local FlatIdent_39764 = 0;
													local Val;
													while true do
														if (FlatIdent_39764 == 0) then
															Val = Indexes[Key];
															return Val[1][Val[2]];
														end
													end
												end,__newindex=function(_, Key, Value)
													local FlatIdent_60A2E = 0;
													local Val;
													while true do
														if (FlatIdent_60A2E == 0) then
															Val = Indexes[Key];
															Val[1][Val[2]] = Value;
															break;
														end
													end
												end});
												FlatIdent_89940 = 2;
											end
										end
									end
								elseif (Enum > 18) then
									Stk[Inst[2]] = Inst[3] ~= 0;
								else
									local FlatIdent_5C540 = 0;
									local A;
									while true do
										if (FlatIdent_5C540 == 3) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_5C540 = 4;
										end
										if (FlatIdent_5C540 == 0) then
											A = nil;
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]];
											FlatIdent_5C540 = 1;
										end
										if (FlatIdent_5C540 == 1) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_5C540 = 2;
										end
										if (FlatIdent_5C540 == 2) then
											Stk[Inst[2]] = Stk[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											A = Inst[2];
											Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
											FlatIdent_5C540 = 3;
										end
										if (6 == FlatIdent_5C540) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											VIP = Inst[3];
											break;
										end
										if (FlatIdent_5C540 == 4) then
											Stk[Inst[2]] = {};
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
											VIP = VIP + 1;
											FlatIdent_5C540 = 5;
										end
										if (5 == FlatIdent_5C540) then
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											FlatIdent_5C540 = 6;
										end
									end
								end
							elseif (Enum <= 22) then
								if (Enum <= 20) then
									local FlatIdent_494DF = 0;
									local A;
									while true do
										if (FlatIdent_494DF == 0) then
											A = Inst[2];
											Stk[A](Stk[A + 1]);
											break;
										end
									end
								elseif (Enum > 21) then
									local A = Inst[2];
									Stk[A] = Stk[A]();
								else
									local Edx;
									local Results;
									local A;
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									A = Inst[2];
									Stk[A](Stk[A + 1]);
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Upvalues[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									A = Inst[2];
									Results = {Stk[A](Stk[A + 1])};
									Edx = 0;
									for Idx = A, Inst[4] do
										Edx = Edx + 1;
										Stk[Idx] = Results[Edx];
									end
									VIP = VIP + 1;
									Inst = Instr[VIP];
									VIP = Inst[3];
								end
							elseif (Enum <= 24) then
								if (Enum > 23) then
									local FlatIdent_3974D = 0;
									while true do
										if (0 == FlatIdent_3974D) then
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_3974D = 1;
										end
										if (FlatIdent_3974D == 4) then
											if (Stk[Inst[2]] ~= Stk[Inst[4]]) then
												VIP = VIP + 1;
											else
												VIP = Inst[3];
											end
											break;
										end
										if (FlatIdent_3974D == 3) then
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_3974D = 4;
										end
										if (2 == FlatIdent_3974D) then
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_3974D = 3;
										end
										if (FlatIdent_3974D == 1) then
											Stk[Inst[2]] = Env[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_3974D = 2;
										end
									end
								else
									local FlatIdent_2CA66 = 0;
									local B;
									local A;
									while true do
										if (FlatIdent_2CA66 == 7) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											VIP = Inst[3];
											break;
										end
										if (FlatIdent_2CA66 == 5) then
											B = Stk[Inst[3]];
											Stk[A + 1] = B;
											Stk[A] = B[Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_2CA66 = 6;
										end
										if (FlatIdent_2CA66 == 0) then
											B = nil;
											A = nil;
											A = Inst[2];
											B = Stk[Inst[3]];
											Stk[A + 1] = B;
											FlatIdent_2CA66 = 1;
										end
										if (4 == FlatIdent_2CA66) then
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											A = Inst[2];
											FlatIdent_2CA66 = 5;
										end
										if (FlatIdent_2CA66 == 3) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											A = Inst[2];
											Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
											VIP = VIP + 1;
											FlatIdent_2CA66 = 4;
										end
										if (FlatIdent_2CA66 == 6) then
											A = Inst[2];
											Stk[A](Stk[A + 1]);
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											FlatIdent_2CA66 = 7;
										end
										if (FlatIdent_2CA66 == 1) then
											Stk[A] = B[Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]];
											VIP = VIP + 1;
											FlatIdent_2CA66 = 2;
										end
										if (FlatIdent_2CA66 == 2) then
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]];
											FlatIdent_2CA66 = 3;
										end
									end
								end
							elseif (Enum > 25) then
								Stk[Inst[2]] = Stk[Inst[3]];
							else
								local FlatIdent_27404 = 0;
								local A;
								while true do
									if (FlatIdent_27404 == 4) then
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_27404 = 5;
									end
									if (FlatIdent_27404 == 7) then
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										break;
									end
									if (FlatIdent_27404 == 3) then
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										FlatIdent_27404 = 4;
									end
									if (FlatIdent_27404 == 2) then
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										FlatIdent_27404 = 3;
									end
									if (FlatIdent_27404 == 5) then
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										FlatIdent_27404 = 6;
									end
									if (FlatIdent_27404 == 6) then
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										FlatIdent_27404 = 7;
									end
									if (FlatIdent_27404 == 0) then
										A = nil;
										A = Inst[2];
										Stk[A](Unpack(Stk, A + 1, Inst[3]));
										VIP = VIP + 1;
										FlatIdent_27404 = 1;
									end
									if (FlatIdent_27404 == 1) then
										Inst = Instr[VIP];
										Stk[Inst[2]] = {};
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_27404 = 2;
									end
								end
							end
						elseif (Enum <= 39) then
							if (Enum <= 32) then
								if (Enum <= 29) then
									if (Enum <= 27) then
										local A = Inst[2];
										local C = Inst[4];
										local CB = A + 2;
										local Result = {Stk[A](Stk[A + 1], Stk[CB])};
										for Idx = 1, C do
											Stk[CB + Idx] = Result[Idx];
										end
										local R = Result[1];
										if R then
											local FlatIdent_7F121 = 0;
											while true do
												if (FlatIdent_7F121 == 0) then
													Stk[CB] = R;
													VIP = Inst[3];
													break;
												end
											end
										else
											VIP = VIP + 1;
										end
									elseif (Enum > 28) then
										if (Stk[Inst[2]] ~= Stk[Inst[4]]) then
											VIP = VIP + 1;
										else
											VIP = Inst[3];
										end
									elseif not Stk[Inst[2]] then
										VIP = VIP + 1;
									else
										VIP = Inst[3];
									end
								elseif (Enum <= 30) then
									local A = Inst[2];
									Stk[A](Unpack(Stk, A + 1, Inst[3]));
								elseif (Enum == 31) then
									local FlatIdent_206F8 = 0;
									local B;
									local A;
									while true do
										if (FlatIdent_206F8 == 6) then
											Stk[A](Unpack(Stk, A + 1, Inst[3]));
											VIP = VIP + 1;
											Inst = Instr[VIP];
											VIP = Inst[3];
											break;
										end
										if (FlatIdent_206F8 == 0) then
											B = nil;
											A = nil;
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											FlatIdent_206F8 = 1;
										end
										if (FlatIdent_206F8 == 2) then
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											A = Inst[2];
											FlatIdent_206F8 = 3;
										end
										if (FlatIdent_206F8 == 3) then
											B = Stk[Inst[3]];
											Stk[A + 1] = B;
											Stk[A] = B[Inst[4]];
											VIP = VIP + 1;
											FlatIdent_206F8 = 4;
										end
										if (5 == FlatIdent_206F8) then
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											A = Inst[2];
											FlatIdent_206F8 = 6;
										end
										if (FlatIdent_206F8 == 1) then
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_206F8 = 2;
										end
										if (4 == FlatIdent_206F8) then
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_206F8 = 5;
										end
									end
								else
									Stk[Inst[2]] = Inst[3];
								end
							elseif (Enum <= 35) then
								if (Enum <= 33) then
									Stk[Inst[2]] = Stk[Inst[3]][Stk[Inst[4]]];
								elseif (Enum == 34) then
									VIP = Inst[3];
								else
									local FlatIdent_56394 = 0;
									while true do
										if (FlatIdent_56394 == 3) then
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_56394 = 4;
										end
										if (FlatIdent_56394 == 0) then
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_56394 = 1;
										end
										if (FlatIdent_56394 == 2) then
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_56394 = 3;
										end
										if (FlatIdent_56394 == 1) then
											Stk[Inst[2]] = Env[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_56394 = 2;
										end
										if (4 == FlatIdent_56394) then
											if (Stk[Inst[2]] ~= Stk[Inst[4]]) then
												VIP = VIP + 1;
											else
												VIP = Inst[3];
											end
											break;
										end
									end
								end
							elseif (Enum <= 37) then
								if (Enum == 36) then
									Stk[Inst[2]][Inst[3]] = Inst[4];
								else
									local A = Inst[2];
									Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
								end
							elseif (Enum == 38) then
								local FlatIdent_6D9D2 = 0;
								local A;
								while true do
									if (FlatIdent_6D9D2 == 1) then
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]];
										VIP = VIP + 1;
										FlatIdent_6D9D2 = 2;
									end
									if (FlatIdent_6D9D2 == 0) then
										A = nil;
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										FlatIdent_6D9D2 = 1;
									end
									if (FlatIdent_6D9D2 == 5) then
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										break;
									end
									if (FlatIdent_6D9D2 == 2) then
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										FlatIdent_6D9D2 = 3;
									end
									if (FlatIdent_6D9D2 == 3) then
										Inst = Instr[VIP];
										A = Inst[2];
										Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
										FlatIdent_6D9D2 = 4;
									end
									if (FlatIdent_6D9D2 == 4) then
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]];
										FlatIdent_6D9D2 = 5;
									end
								end
							else
								local FlatIdent_571C2 = 0;
								while true do
									if (FlatIdent_571C2 == 0) then
										Stk[Inst[2]] = Upvalues[Inst[3]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										FlatIdent_571C2 = 1;
									end
									if (FlatIdent_571C2 == 2) then
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_571C2 = 3;
									end
									if (FlatIdent_571C2 == 3) then
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										if (Stk[Inst[2]] == Stk[Inst[4]]) then
											VIP = VIP + 1;
										else
											VIP = Inst[3];
										end
										break;
									end
									if (FlatIdent_571C2 == 1) then
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Env[Inst[3]];
										VIP = VIP + 1;
										FlatIdent_571C2 = 2;
									end
								end
							end
						elseif (Enum <= 46) then
							if (Enum <= 42) then
								if (Enum <= 40) then
									local FlatIdent_20FE3 = 0;
									local B;
									local A;
									while true do
										if (FlatIdent_20FE3 == 0) then
											B = nil;
											A = nil;
											A = Inst[2];
											FlatIdent_20FE3 = 1;
										end
										if (8 == FlatIdent_20FE3) then
											Inst = Instr[VIP];
											A = Inst[2];
											B = Stk[Inst[3]];
											FlatIdent_20FE3 = 9;
										end
										if (FlatIdent_20FE3 == 7) then
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											FlatIdent_20FE3 = 8;
										end
										if (FlatIdent_20FE3 == 5) then
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_20FE3 = 6;
										end
										if (2 == FlatIdent_20FE3) then
											Stk[Inst[2]] = Env[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_20FE3 = 3;
										end
										if (FlatIdent_20FE3 == 3) then
											A = Inst[2];
											B = Stk[Inst[3]];
											Stk[A + 1] = B;
											FlatIdent_20FE3 = 4;
										end
										if (FlatIdent_20FE3 == 4) then
											Stk[A] = B[Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_20FE3 = 5;
										end
										if (FlatIdent_20FE3 == 1) then
											Stk[A](Unpack(Stk, A + 1, Inst[3]));
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_20FE3 = 2;
										end
										if (6 == FlatIdent_20FE3) then
											A = Inst[2];
											Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
											VIP = VIP + 1;
											FlatIdent_20FE3 = 7;
										end
										if (FlatIdent_20FE3 == 9) then
											Stk[A + 1] = B;
											Stk[A] = B[Inst[4]];
											break;
										end
									end
								elseif (Enum > 41) then
									Stk[Inst[2]] = Upvalues[Inst[3]];
								else
									local A = Inst[2];
									local Cls = {};
									for Idx = 1, #Lupvals do
										local FlatIdent_44265 = 0;
										local List;
										while true do
											if (0 == FlatIdent_44265) then
												List = Lupvals[Idx];
												for Idz = 0, #List do
													local Upv = List[Idz];
													local NStk = Upv[1];
													local DIP = Upv[2];
													if ((NStk == Stk) and (DIP >= A)) then
														Cls[DIP] = NStk[DIP];
														Upv[1] = Cls;
													end
												end
												break;
											end
										end
									end
								end
							elseif (Enum <= 44) then
								if (Enum == 43) then
									local FlatIdent_15A17 = 0;
									local A;
									while true do
										if (24 == FlatIdent_15A17) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_15A17 = 25;
										end
										if (FlatIdent_15A17 == 27) then
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											FlatIdent_15A17 = 28;
										end
										if (FlatIdent_15A17 == 7) then
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											FlatIdent_15A17 = 8;
										end
										if (FlatIdent_15A17 == 6) then
											Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Env[Inst[3]];
											VIP = VIP + 1;
											FlatIdent_15A17 = 7;
										end
										if (FlatIdent_15A17 == 14) then
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											A = Inst[2];
											FlatIdent_15A17 = 15;
										end
										if (FlatIdent_15A17 == 25) then
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											A = Inst[2];
											Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
											FlatIdent_15A17 = 26;
										end
										if (FlatIdent_15A17 == 8) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_15A17 = 9;
										end
										if (22 == FlatIdent_15A17) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											A = Inst[2];
											Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
											VIP = VIP + 1;
											FlatIdent_15A17 = 23;
										end
										if (FlatIdent_15A17 == 0) then
											A = nil;
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
											FlatIdent_15A17 = 1;
										end
										if (FlatIdent_15A17 == 20) then
											Stk[Inst[2]] = Env[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											FlatIdent_15A17 = 21;
										end
										if (FlatIdent_15A17 == 15) then
											Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Inst[4];
											VIP = VIP + 1;
											FlatIdent_15A17 = 16;
										end
										if (FlatIdent_15A17 == 4) then
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											FlatIdent_15A17 = 5;
										end
										if (FlatIdent_15A17 == 2) then
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											A = Inst[2];
											Stk[A] = Stk[A](Stk[A + 1]);
											FlatIdent_15A17 = 3;
										end
										if (FlatIdent_15A17 == 12) then
											Inst = Instr[VIP];
											A = Inst[2];
											Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_15A17 = 13;
										end
										if (10 == FlatIdent_15A17) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Env[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_15A17 = 11;
										end
										if (5 == FlatIdent_15A17) then
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											A = Inst[2];
											FlatIdent_15A17 = 6;
										end
										if (FlatIdent_15A17 == 21) then
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											FlatIdent_15A17 = 22;
										end
										if (FlatIdent_15A17 == 19) then
											Inst = Instr[VIP];
											A = Inst[2];
											Stk[A] = Stk[A](Stk[A + 1]);
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_15A17 = 20;
										end
										if (FlatIdent_15A17 == 11) then
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											FlatIdent_15A17 = 12;
										end
										if (18 == FlatIdent_15A17) then
											Stk[Inst[2]] = Env[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											FlatIdent_15A17 = 19;
										end
										if (FlatIdent_15A17 == 29) then
											Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Env[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											FlatIdent_15A17 = 30;
										end
										if (FlatIdent_15A17 == 17) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Inst[4];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_15A17 = 18;
										end
										if (FlatIdent_15A17 == 23) then
											Inst = Instr[VIP];
											Stk[Inst[2]] = Env[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											FlatIdent_15A17 = 24;
										end
										if (FlatIdent_15A17 == 16) then
											Inst = Instr[VIP];
											Stk[Inst[2]] = Env[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											FlatIdent_15A17 = 17;
										end
										if (FlatIdent_15A17 == 13) then
											Stk[Inst[2]] = Env[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											FlatIdent_15A17 = 14;
										end
										if (FlatIdent_15A17 == 9) then
											A = Inst[2];
											Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Inst[4];
											FlatIdent_15A17 = 10;
										end
										if (FlatIdent_15A17 == 3) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Env[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_15A17 = 4;
										end
										if (FlatIdent_15A17 == 28) then
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											A = Inst[2];
											Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_15A17 = 29;
										end
										if (FlatIdent_15A17 == 30) then
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											break;
										end
										if (FlatIdent_15A17 == 1) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Inst[4];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Env[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_15A17 = 2;
										end
										if (FlatIdent_15A17 == 26) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]][Inst[3]] = Inst[4];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Env[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_15A17 = 27;
										end
									end
								else
									for Idx = Inst[2], Inst[3] do
										Stk[Idx] = nil;
									end
								end
							elseif (Enum > 45) then
								do
									return;
								end
							else
								Stk[Inst[2]] = Stk[Inst[3]] - Stk[Inst[4]];
							end
						elseif (Enum <= 49) then
							if (Enum <= 47) then
								Stk[Inst[2]][Stk[Inst[3]]] = Stk[Inst[4]];
							elseif (Enum > 48) then
								local FlatIdent_90A69 = 0;
								while true do
									if (FlatIdent_90A69 == 7) then
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_90A69 = 8;
									end
									if (FlatIdent_90A69 == 1) then
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_90A69 = 2;
									end
									if (FlatIdent_90A69 == 8) then
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_90A69 = 9;
									end
									if (FlatIdent_90A69 == 3) then
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_90A69 = 4;
									end
									if (FlatIdent_90A69 == 4) then
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_90A69 = 5;
									end
									if (FlatIdent_90A69 == 2) then
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_90A69 = 3;
									end
									if (FlatIdent_90A69 == 0) then
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_90A69 = 1;
									end
									if (5 == FlatIdent_90A69) then
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_90A69 = 6;
									end
									if (FlatIdent_90A69 == 6) then
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_90A69 = 7;
									end
									if (FlatIdent_90A69 == 9) then
										Stk[Inst[2]][Inst[3]] = Inst[4];
										break;
									end
								end
							else
								local A;
								local K;
								local B;
								Stk[Inst[2]] = Stk[Inst[3]];
								VIP = VIP + 1;
								Inst = Instr[VIP];
								Stk[Inst[2]] = Inst[3];
								VIP = VIP + 1;
								Inst = Instr[VIP];
								Stk[Inst[2]] = Stk[Inst[3]];
								VIP = VIP + 1;
								Inst = Instr[VIP];
								B = Inst[3];
								K = Stk[B];
								for Idx = B + 1, Inst[4] do
									K = K .. Stk[Idx];
								end
								Stk[Inst[2]] = K;
								VIP = VIP + 1;
								Inst = Instr[VIP];
								A = Inst[2];
								Stk[A](Stk[A + 1]);
							end
						elseif (Enum <= 51) then
							if (Enum > 50) then
								local FlatIdent_5C6FB = 0;
								local A;
								local B;
								while true do
									if (FlatIdent_5C6FB == 0) then
										A = Inst[2];
										B = Stk[Inst[3]];
										FlatIdent_5C6FB = 1;
									end
									if (FlatIdent_5C6FB == 1) then
										Stk[A + 1] = B;
										Stk[A] = B[Inst[4]];
										break;
									end
								end
							else
								local FlatIdent_508D4 = 0;
								local B;
								local A;
								while true do
									if (FlatIdent_508D4 == 17) then
										Inst = Instr[VIP];
										A = Inst[2];
										Stk[A](Stk[A + 1]);
										VIP = VIP + 1;
										Inst = Instr[VIP];
										do
											return;
										end
										break;
									end
									if (FlatIdent_508D4 == 16) then
										Stk[Inst[2]] = Upvalues[Inst[3]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										A = Inst[2];
										B = Stk[Inst[3]];
										Stk[A + 1] = B;
										Stk[A] = B[Inst[4]];
										VIP = VIP + 1;
										FlatIdent_508D4 = 17;
									end
									if (FlatIdent_508D4 == 0) then
										B = nil;
										A = nil;
										Stk[Inst[2]] = Upvalues[Inst[3]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Upvalues[Inst[3]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_508D4 = 1;
									end
									if (FlatIdent_508D4 == 2) then
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										FlatIdent_508D4 = 3;
									end
									if (FlatIdent_508D4 == 15) then
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										A = Inst[2];
										Stk[A](Stk[A + 1]);
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_508D4 = 16;
									end
									if (FlatIdent_508D4 == 14) then
										VIP = VIP + 1;
										Inst = Instr[VIP];
										A = Inst[2];
										Stk[A](Unpack(Stk, A + 1, Inst[3]));
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Env[Inst[3]];
										VIP = VIP + 1;
										FlatIdent_508D4 = 15;
									end
									if (11 == FlatIdent_508D4) then
										Inst = Instr[VIP];
										A = Inst[2];
										Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_508D4 = 12;
									end
									if (5 == FlatIdent_508D4) then
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Env[Inst[3]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_508D4 = 6;
									end
									if (FlatIdent_508D4 == 13) then
										Inst = Instr[VIP];
										Stk[Inst[2]] = Env[Inst[3]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										FlatIdent_508D4 = 14;
									end
									if (FlatIdent_508D4 == 4) then
										Inst = Instr[VIP];
										Stk[Inst[2]] = Env[Inst[3]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										FlatIdent_508D4 = 5;
									end
									if (FlatIdent_508D4 == 7) then
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										A = Inst[2];
										Stk[A](Stk[A + 1]);
										VIP = VIP + 1;
										FlatIdent_508D4 = 8;
									end
									if (FlatIdent_508D4 == 10) then
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										FlatIdent_508D4 = 11;
									end
									if (FlatIdent_508D4 == 6) then
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										A = Inst[2];
										Stk[A](Unpack(Stk, A + 1, Inst[3]));
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Env[Inst[3]];
										FlatIdent_508D4 = 7;
									end
									if (FlatIdent_508D4 == 12) then
										Stk[Inst[2]] = Env[Inst[3]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										FlatIdent_508D4 = 13;
									end
									if (FlatIdent_508D4 == 9) then
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_508D4 = 10;
									end
									if (FlatIdent_508D4 == 3) then
										VIP = VIP + 1;
										Inst = Instr[VIP];
										A = Inst[2];
										Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										FlatIdent_508D4 = 4;
									end
									if (FlatIdent_508D4 == 8) then
										Inst = Instr[VIP];
										Stk[Inst[2]] = Upvalues[Inst[3]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Upvalues[Inst[3]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Env[Inst[3]];
										FlatIdent_508D4 = 9;
									end
									if (FlatIdent_508D4 == 1) then
										Stk[Inst[2]] = Env[Inst[3]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										FlatIdent_508D4 = 2;
									end
								end
							end
						elseif (Enum == 52) then
							if (Stk[Inst[2]] == Stk[Inst[4]]) then
								VIP = VIP + 1;
							else
								VIP = Inst[3];
							end
						else
							local FlatIdent_74B46 = 0;
							local B;
							local A;
							while true do
								if (23 == FlatIdent_74B46) then
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									A = Inst[2];
									Stk[A] = Stk[A]();
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									FlatIdent_74B46 = 24;
								end
								if (FlatIdent_74B46 == 2) then
									Inst = Instr[VIP];
									A = Inst[2];
									Stk[A] = Stk[A](Stk[A + 1]);
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									FlatIdent_74B46 = 3;
								end
								if (FlatIdent_74B46 == 13) then
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Inst[4];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
									VIP = VIP + 1;
									FlatIdent_74B46 = 14;
								end
								if (FlatIdent_74B46 == 8) then
									Inst = Instr[VIP];
									A = Inst[2];
									Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									FlatIdent_74B46 = 9;
								end
								if (18 == FlatIdent_74B46) then
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									FlatIdent_74B46 = 19;
								end
								if (FlatIdent_74B46 == 16) then
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									FlatIdent_74B46 = 17;
								end
								if (FlatIdent_74B46 == 9) then
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									FlatIdent_74B46 = 10;
								end
								if (22 == FlatIdent_74B46) then
									VIP = VIP + 1;
									Inst = Instr[VIP];
									A = Inst[2];
									Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Inst[4];
									FlatIdent_74B46 = 23;
								end
								if (FlatIdent_74B46 == 0) then
									B = nil;
									A = nil;
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									A = Inst[2];
									Stk[A] = Stk[A]();
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Inst[4];
									FlatIdent_74B46 = 1;
								end
								if (12 == FlatIdent_74B46) then
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									A = Inst[2];
									Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
									FlatIdent_74B46 = 13;
								end
								if (FlatIdent_74B46 == 5) then
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									A = Inst[2];
									FlatIdent_74B46 = 6;
								end
								if (FlatIdent_74B46 == 15) then
									A = Inst[2];
									Stk[A] = Stk[A](Stk[A + 1]);
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									FlatIdent_74B46 = 16;
								end
								if (FlatIdent_74B46 == 14) then
									Inst = Instr[VIP];
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									FlatIdent_74B46 = 15;
								end
								if (FlatIdent_74B46 == 3) then
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									A = Inst[2];
									B = Stk[Inst[3]];
									Stk[A + 1] = B;
									Stk[A] = B[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									FlatIdent_74B46 = 4;
								end
								if (FlatIdent_74B46 == 1) then
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									FlatIdent_74B46 = 2;
								end
								if (FlatIdent_74B46 == 21) then
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									FlatIdent_74B46 = 22;
								end
								if (FlatIdent_74B46 == 10) then
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									A = Inst[2];
									Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
									VIP = VIP + 1;
									Inst = Instr[VIP];
									FlatIdent_74B46 = 11;
								end
								if (FlatIdent_74B46 == 7) then
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									FlatIdent_74B46 = 8;
								end
								if (FlatIdent_74B46 == 20) then
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Inst[4];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									FlatIdent_74B46 = 21;
								end
								if (17 == FlatIdent_74B46) then
									VIP = VIP + 1;
									Inst = Instr[VIP];
									A = Inst[2];
									Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Env[Inst[3]];
									FlatIdent_74B46 = 18;
								end
								if (FlatIdent_74B46 == 4) then
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									A = Inst[2];
									Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									FlatIdent_74B46 = 5;
								end
								if (FlatIdent_74B46 == 11) then
									Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									FlatIdent_74B46 = 12;
								end
								if (FlatIdent_74B46 == 19) then
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									A = Inst[2];
									Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
									VIP = VIP + 1;
									FlatIdent_74B46 = 20;
								end
								if (6 == FlatIdent_74B46) then
									Stk[A] = Stk[A](Stk[A + 1]);
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									FlatIdent_74B46 = 7;
								end
								if (FlatIdent_74B46 == 24) then
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									FlatIdent_74B46 = 25;
								end
								if (FlatIdent_74B46 == 25) then
									Inst = Instr[VIP];
									A = Inst[2];
									B = Stk[Inst[3]];
									Stk[A + 1] = B;
									Stk[A] = B[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									break;
								end
							end
						end
						VIP = VIP + 1;
						break;
					end
				end
			end
		end;
	end
	return Wrap(Deserialize(), {}, vmenv)(...);
end
return VMCall("LOL!863O0003073O0067657467656E7603103O004E6F74696669636174696F6E5465787403283O0054797065202F636F6E736F6C6520746F20732O6520746865206C697374206F6620776F7264203B2903083O00496E7374616E63652O033O006E657703093O005363722O656E47756903063O00506172656E7403043O0067616D6503073O00506C6179657273030B3O004C6F63616C506C61796572030C3O0057616974466F724368696C6403093O00506C6179657247756903053O004672616D6503043O0053697A6503053O005544696D32028O00025O00C07240026O00594003083O00506F736974696F6E026O00E03F025O00C062C0026O0059C003103O004261636B67726F756E64436F6C6F723303063O00436F6C6F723303073O0066726F6D524742030F3O00426F7264657253697A65506978656C03093O00546578744C6162656C026O00F03F03163O004261636B67726F756E645472616E73706172656E6379030A3O0054657874436F6C6F7233025O00E06F40030A3O00546578745363616C65642O0103043O0054657874030A3O0047657453657276696365030C3O0054772O656E5365727669636503043O004E616D6503353O004E657074756E6520427970612O736572206D616E75616C20286D61646520627920646162752073686F776361736520696E20797429026O007940026O006940026O0069C0026O004940033A3O004E657074756E6520627970612O736572206D616E75616C20286D61646520627920646162752073686F776361736520646F6E277420736B696429026O003E4003043O00466F6E7403043O00456E756D030A3O00476F7468616D426F6C6403083O005465787453697A65026O00344003073O0054657874426F78029A5O99E93F026O33D33F029A5O99B93F026O66D63F030F3O00506C616365686F6C64657254657874030F3O00456E746572206120776F72643O2E03063O00476F7468616D026O003240030A3O005465787442752O746F6E029A5O99C93F026O66E63F030B3O004175746F636F2O72656374025O00805140030A3O00496E707574426567616E03073O00436F2O6E656374030C3O00496E7075744368616E67656403103O0055736572496E7075745365727669636503043O006675636B03133O00D293CCB2E1B49CCCB2E1B484CCB2E1B48BCCB203043O00636F636B03143O00E1B484CCB2E1B48FCCB2E1B484CCB2E1B48BCCB203053O006E692O676103153O00C9B4CCB2C9AACCB2C9A2CCB2C9A2CCB2E1B480CCB203063O0066656D626F79031B3O00D293CCB2E1B487CCB2E1B48DCCB2CA99CCB2E1B48FCCB2CA8FCCB22O033O00676179030D3O00C9A2CCB2E1B480CCB2CA8FCCB203043O00706F726E03123O00E1B498CCB2E1B48FCCB2CA80CCB2C9B4CCB203093O007065646F7068696C65032A3O00E1B498CCB2E1B487CCB2E1B485CCB2E1B48FCCB2E1B498CCB2CA9CCCB2C9AACCB2CA9FCCB2E1B487CCB203043O007368697403103O0073CCB2CA9CCCB2C9AACCB2E1B49BCCB22O033O0063756D030F3O00E1B484CCB2E1B49CCCB2E1B48DCCB203053O00626974636803163O00CA99CCB2C9AACCB2E1B49BCCB2E1B484CCB2CA9CCCB203053O0070752O737903143O00E1B498CCB2E1B49CCCB273CCB273CCB2CA8FCCB22O033O00736578033D3O0073CCB3D0B5CCB3E281A5CCB3E281A5CCB3E281A5CCB3E281A5CCB3E281A5CCB3E281A5CCB3E281A5CCB3E281A5CCB3E281A5CCB3E281A5CCB3D185CCB303043O00736C757403113O0073CC85CA9FCC85E1B49CCC85E1B49BCC8503043O0063756E7403133O00E1B484CCB2E1B49CCCB2C9B4CCB2E1B49BCCB203043O007469747303113O00E1B49BCCB2C9AACCB2E1B49BCCB273CCB22O033O006B7973030C3O00E1B48BCCB2CA8FCCB273CCB203043O006469636B03133O00E1B485CCB2C9AACCB2E1B484CCB2E1B48BCCB203053O00622O6F627303153O00CA99CCB2E1B48FCCB2E1B48FCCB2CA99CCB273CCB22O033O00612O73030B3O00E1B480CCB273CCB273CCB203073O006675636B696E67031F3O00D293CCB2E1B49CCCB2E1B484CCB2E1B48BCCB2C9AACCB2C9B4CCB2C9A2CCB203063O006D696E6F727303173O004DCCB2C9AACCB2C9B4CCB2E1B48FCCB2CA80CCB273CCB22O033O00686F65030E3O00CA9CCCB2E1B48FCCB2E1B487CCB203053O00686F726E7903153O00CA9CCCB2E1B48FCCB2CA80CCB2C9B4CCB2CA8FCCB203063O00726574617264031C3O00CA80CCB2E1B487CCB2E1B49BCCB2E1B480CCB2CA80CCB2E1B485CCB203063O0066612O676F74031B3O00D293CCB2E1B480CCB2C9A2CCB2C9A2CCB2E1B48FCCB2E1B49BCCB2030A3O006D617374757262617465032E3O00E1B48DCCB2E1B480CCB273CCB2E1B49BCCB2E1B49CCCB2CA80CCB2CA99CCB2E1B480CCB2E1B49BCCB2E1B487CCB203053O00736C61766503163O0073CCB2CA9FCCB2E1B480CCB2E1B4A0CCB2E1B487CCB203063O00736C6176657303193O0073CCB2CA9FCCB2E1B480CCB2E1B4A0CCB2E1B487CCB273CCB203053O006E616B656403183O00C9B4CCB2E1B480CCB2E1B48BCCB2E1B487CCB2E1B485CCB203073O006675636B657273031F3O00D293CCB2E1B49CCCB2E1B484CCB2E1B48BCCB2E1B487CCB2CA80CCB273CCB203063O00736C752O7479031A3O0073CC85CA9FCC85E1B49CCC85E1B49BCC85E1B49BCC85CA8FCC8503063O0074692O746573031B3O00E1B49BCCB2C9AACCB2E1B49BCCB2E1B49BCCB2E1B487CCB273CCB203063O006E692O67657203193O00C9B4CCB2C9AACCB2C9A2CCB2C9A2CCB2E1B487CCB2CA80CCB203113O004D6F75736542752O746F6E31436C69636B004A012O0012353O00018O0001000200304O0002000300124O00043O00206O000500122O000100068O0002000200122O000100083O00202O00010001000900202O00010001000A00202O00010001000B00122O0003000C6O00010003000200104O0007000100122O000100043O00202O00010001000500122O0002000D6O00010002000200122O0002000F3O00202O00020002000500122O000300103O00122O000400113O00122O000500103O00122O000600126O00020006000200102O0001000E000200122O0002000F3O00202O00020002000500122O000300143O00122O000400153O00122O000500103O00122O000600166O00020006000200102O00010013000200122O000200183O00202O00020002001900122O000300103O00122O000400103O00122O000500106O00020005000200102O00010017000200302O0001001A001000102O000100073O00122O000200043O00202O00020002000500122O0003001B6O00020002000200122O0003000F3O00202O00030003000500122O0004001C3O00122O000500103O00122O0006001C3O00122O000700106O00030007000200102O0002000E000300122O0003000F3O00202O00030003000500122O000400103O00122O000500103O00122O000600103O00122O000700106O00030007000200102O00020013000300302O0002001D001C00122O000300183O00202O00030003001900122O0004001F3O00122O0005001F3O00122O0006001F6O00030006000200102O0002001E000300302O00020020002100122O000300016O00030001000200202O00030003000200102O00020022000300102O00020007000100122O000300083O00202O00030003002300122O000500244O002500030005000200061000043O000100012O001A3O00033O00061000050001000100022O001A3O00044O001A3O00014O0005000600056O00060001000100122O000600043O00202O00060006000500122O000700066O00060002000200122O000700083O00202O00070007000900202O00070007000A00202O00070007000B00122O0009000C6O00070009000200102O00060007000700302O00060025002600122O000700043O00202O00070007000500122O0008000D6O00070002000200102O00070007000600122O0008000F3O00202O00080008000500122O000900103O00122O000A00273O00122O000B00103O00122O000C00286O0008000C000200102O0007000E000800122O0008000F3O00202O00080008000500122O000900143O00122O000A00293O00122O000B00143O00122O000C00166O0008000C000200102O00070013000800122O000800183O00202O00080008001900122O0009002A3O00122O000A002A3O00122O000B002A6O0008000B000200102O00070017000800302O0007001A001000122O000800043O00202O00080008000500122O0009001B6O00080002000200102O00080007000700122O0009000F3O00202O00090009000500122O000A001C3O00122O000B00103O00122O000C00103O00122O000D002A6O0009000D000200102O0008000E000900122O0009000F3O00202O00090009000500122O000A00103O00122O000B00103O00122O000C00103O00122O000D00106O0009000D000200102O00080013000900302O00080022002B00122O000900183O00202O00090009001900122O000A001F3O00122O000B001F3O00122O000C001F6O0009000C000200102O0008001E000900122O000900183O00202O00090009001900122O000A002C3O00122O000B002C3O00122O000C002C6O0009000C000200102O00080017000900302O0008001A001000120B0009002E3O00202B00090009002D00202O00090009002F00102O0008002D000900302O00080030003100122O000900043O00202O00090009000500122O000A00326O00090002000200102O00090007000700122O000A000F3O00202O000A000A000500122O000B00333O00122O000C00103O00122O000D00343O00122O000E00106O000A000E000200102O0009000E000A00122O000A000F3O00202O000A000A000500122O000B00353O00122O000C00103O00122O000D00363O00122O000E00106O000A000E000200102O00090013000A00302O00090037003800122O000A00183O00202O000A000A001900122O000B00103O00122O000C00103O00122O000D00106O000A000D000200102O0009001E000A00122O000A00183O00202O000A000A001900122O000B001F3O00122O000C001F3O00122O000D001F6O000A000D000200102O00090017000A00302O0009001A001000122O000A002E3O00202O000A000A002D00202O000A000A003900102O0009002D000A00302O00090030003A00122O000A00043O00202O000A000A000500122O000B003B6O000A0002000200102O000A0007000700122O000B000F3O00202O000B000B000500122O000C00333O00122O000D00103O00122O000E003C3O00122O000F00106O000B000F000200102O000A000E000B00122O000B000F3O00202O000B000B000500122O000C00353O00122O000D00103O00122O000E003D3O00122O000F00106O000B000F000200102O000A0013000B00302O000A0022003E00122O000B00183O00202O000B000B001900122O000C001F3O00122O000D001F3O00122O000E001F6O000B000E000200102O000A001E000B00122O000B00183O00202O000B000B001900122O000C003F3O00122O000D003F3O00122O000E003F4O000C000B000E000200102O000A0017000B00302O000A001A001000122O000B002E3O00202O000B000B002D00202O000B000B002F00102O000A002D000B00302O000A0030003A4O000B000E3O000610000F0002000100032O001A3O000D4O001A3O00074O001A3O000E3O00200900100007004000203300100010004100061000120003000100042O001A3O000B4O001A3O000D4O001A3O000E4O001A3O00074O001E00100012000100200900100007004200203300100010004100061000120004000100012O001A3O000C4O002800100012000100122O001000083O00202O00100010002300122O001200436O00100012000200202O00100010004200202O00100010004100061000120005000100032O001A3O000C4O001A3O000B4O001A3O000F4O00190010001200014O00103O001900302O00100044004500302O00100046004700302O00100048004900302O0010004A004B00302O0010004C004D00302O0010004E004F00302O00100050005100302O00100052005300300200100054005500302O00100056005700302O00100058005900302O0010005A005B00302O0010005C005D00302O0010005E005F00302O00100060006100302O00100062006300302O00100064006500302O00100066006700300200100068006900302O0010006A006B00302O0010006C006D00302O0010006E006F00302O00100070007100302O00100072007300302O00100074007500302O00100076007700302O00100078007900302O0010007A007B0030240010007C007D0030240010007E007F00302400100080008100302400100082008300302400100084008500061000110006000100012O001A3O00104O001A001200114O00030012000100010020090012000A008600203300120012004100061000140007000100022O001A3O00104O001A3O00094O001E0012001400012O00298O002E3O00013O00083O000A3O00028O00026O00F03F03093O0054772O656E496E666F2O033O006E657703083O00506F736974696F6E027O004003093O00436F6D706C6574656403043O005761697403063O0043726561746503043O00506C617905333O001220000500014O002C000600083O00260100050017000100010004223O00170001001220000900013O00260100090009000100020004223O00090001001220000500023O0004223O0017000100260100090005000100010004223O0005000100120B000A00033O002012000A000A00044O000B00026O000C00036O000D00046O000A000D00024O0006000A6O000A3O000100102O000A000500014O0007000A3O00122O000900023O00044O000500010026010005001D000100060004223O001D00010020090009000800070020330009000900082O00140009000200010004223O0032000100260100050002000100020004223O00020001001220000900013O00260100090024000100020004223O00240001001220000500063O0004223O0002000100260100090020000100010004223O002000012O002A000A5O002017000A000A00094O000C8O000D00066O000E00076O000A000E00024O0008000A3O00202O000A0008000A4O000A0002000100122O000900023O00044O002000010004223O000200012O002E3O00017O00103O0003053O005544696D322O033O006E6577026O00E03F025O00C062C0026O0049C003043O00456E756D030B3O00456173696E675374796C6503043O0053696E65030F3O00456173696E67446972656374696F6E2O033O004F757403043O0077616974027O0040026O00F03F026O00594003023O00496E03073O0044657374726F79002C4O00329O00000100013O00122O000200013O00202O00020002000200122O000300033O00122O000400043O00122O000500033O00122O000600056O00020006000200122O000300033O00122O000400063O00202O00040004000700202O00040004000800122O000500063O00202O00050005000900202O00050005000A6O0005000100124O000B3O00122O0001000C8O000200019O004O000100013O00122O000200013O00202O00020002000200122O000300033O00122O000400043O00122O0005000D3O00122O0006000E6O00020006000200122O000300033O00122O000400063O00202O00040004000700202O00040004000800122O000500063O00202O00050005000900202O00050005000F6O0005000100124O000B3O00122O000100038O000200016O00013O00206O00106O000200016O00017O00083O00028O0003083O00506F736974696F6E03053O005544696D322O033O006E657703013O005803053O005363616C6503063O004F2O6673657403013O0059011F3O001220000100014O002C000200023O00260100010002000100010004223O0002000100200900033O00024O00048O0002000300044O000300013O00122O000400033O00202O0004000400044O000500023O00202O00050005000500202O0005000500064O000600023O00202O00060006000500202O00060006000700202O0007000200054O0006000600074O000700023O00202O00070007000800202O0007000700064O000800023O00202O00080008000800202O00080008000700202O0009000200084O0008000800094O00040008000200102O00030002000400044O001E00010004223O000200012O002E3O00017O00093O00030D3O0055736572496E7075745479706503043O00456E756D030C3O004D6F75736542752O746F6E3103053O00546F756368028O0003083O00506F736974696F6E026O00F03F03073O004368616E67656403073O00436F2O6E65637401283O00201800013O000100122O000200023O00202O00020002000100202O00020002000300062O0001000C000100020004223O000C000100200900013O000100120B000200023O00200900020002000100200900020002000400063400010027000100020004223O00270001001220000100054O002C000200023O0026010001000E000100050004223O000E0001001220000200053O00260100020018000100050004223O001800012O0013000300014O000400035O00200900033O00062O0004000300013O001220000200073O00260100020011000100070004223O001100012O002A000300033O0020090003000300062O0004000300023O00200900033O000800203300030003000900061000053O000100022O001A8O002A8O001E0003000500010004223O002700010004223O001100010004223O002700010004223O000E00012O002E3O00013O00013O00033O00030E3O0055736572496E707574537461746503043O00456E756D2O033O00456E64000A4O00277O00206O000100122O000100023O00202O00010001000100202O00010001000300064O0009000100010004223O000900012O00138O00043O00014O002E3O00017O00043O00030D3O0055736572496E7075745479706503043O00456E756D030D3O004D6F7573654D6F76656D656E7403053O00546F756368010E3O00201800013O000100122O000200023O00202O00020002000100202O00020002000300062O0001000C000100020004223O000C000100200900013O000100120B000200023O0020090002000200010020090002000200040006340001000D000100020004223O000D00012O00048O002E3O00019O002O00010A4O002A00015O0006343O0009000100010004223O000900012O002A000100013O0006070001000900013O0004223O000900012O002A000100024O001A00026O00140001000200012O002E3O00017O00053O00028O0003053O007072696E7403113O004175746F636F2O72656374204C6973743A03053O00706169727303043O00202D3E2000153O0012203O00013O0026013O0001000100010004223O0001000100120B000100023O001215000200036O00010002000100122O000100046O00028O00010002000300044O0010000100120B000600024O0030000700043O00122O000800056O000900056O0007000700094O00060002000100061B0001000A000100020004223O000A00010004223O001400010004223O000100012O002E3O00017O00103O00028O00026O00F03F03063O0069706169727303053O007461626C6503063O00636F6E63617403013O0020027O004003043O0067616D6503113O005265706C69636174656453746F72616765031B3O0044656661756C744368617453797374656D436861744576656E747303113O005361794D652O7361676552657175657374030A3O00466972655365727665722O033O00412O6C03043O005465787403053O006C6F77657203053O0073706C697400303O0012203O00014O002C000100033O0026013O0017000100020004223O0017000100120B000400034O001A000500024O00060004000200060004223O000E00012O002A00096O002100090009000800061C0009000D000100010004223O000D00012O001A000900084O002F00020007000900061B00040008000100020004223O0008000100120B000400043O0020260004000400054O000500023O00122O000600066O0004000600024O000300043O00124O00073O0026013O0022000100070004223O0022000100120B000400083O00201F00040004000900202O00040004000A00202O00040004000B00202O00040004000C4O000600033O00122O0007000D6O00040007000100044O002F00010026013O0002000100010004223O000200012O002A000400013O00200F00040004000E00202O00040004000F4O0004000200024O000100043O00202O00040001001000122O000600066O0004000600024O000200043O00124O00023O00044O000200012O002E3O00017O00", GetFEnv(), ...);
