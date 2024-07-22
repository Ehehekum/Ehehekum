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
			local FlatIdent_3CDED = 0;
			while true do
				if (0 == FlatIdent_3CDED) then
					repeatNext = StrToNumber(Sub(byte, 1, 1));
					return "";
				end
			end
		else
			local a = Char(StrToNumber(byte, 16));
			if repeatNext then
				local b = Rep(a, repeatNext);
				repeatNext = nil;
				return b;
			else
				return a;
			end
		end
	end);
	local function gBit(Bit, Start, End)
		if End then
			local FlatIdent_76979 = 0;
			local Res;
			while true do
				if (FlatIdent_76979 == 0) then
					Res = (Bit / (2 ^ (Start - 1))) % (2 ^ (((End - 1) - (Start - 1)) + 1));
					return Res - (Res % 1);
				end
			end
		else
			local Plc = 2 ^ (Start - 1);
			return (((Bit % (Plc + Plc)) >= Plc) and 1) or 0;
		end
	end
	local function gBits8()
		local FlatIdent_21CA5 = 0;
		local a;
		while true do
			if (FlatIdent_21CA5 == 0) then
				a = Byte(ByteString, DIP, DIP);
				DIP = DIP + 1;
				FlatIdent_21CA5 = 1;
			end
			if (FlatIdent_21CA5 == 1) then
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
		local FlatIdent_803FB = 0;
		local a;
		local b;
		local c;
		local d;
		while true do
			if (FlatIdent_803FB == 1) then
				return (d * 16777216) + (c * 65536) + (b * 256) + a;
			end
			if (FlatIdent_803FB == 0) then
				a, b, c, d = Byte(ByteString, DIP, DIP + 3);
				DIP = DIP + 4;
				FlatIdent_803FB = 1;
			end
		end
	end
	local function gFloat()
		local FlatIdent_10DED = 0;
		local Left;
		local Right;
		local IsNormal;
		local Mantissa;
		local Exponent;
		local Sign;
		while true do
			if (FlatIdent_10DED == 1) then
				IsNormal = 1;
				Mantissa = (gBit(Right, 1, 20) * (2 ^ 32)) + Left;
				FlatIdent_10DED = 2;
			end
			if (2 == FlatIdent_10DED) then
				Exponent = gBit(Right, 21, 31);
				Sign = ((gBit(Right, 32) == 1) and -1) or 1;
				FlatIdent_10DED = 3;
			end
			if (FlatIdent_10DED == 0) then
				Left = gBits32();
				Right = gBits32();
				FlatIdent_10DED = 1;
			end
			if (3 == FlatIdent_10DED) then
				if (Exponent == 0) then
					if (Mantissa == 0) then
						return Sign * 0;
					else
						local FlatIdent_67F21 = 0;
						while true do
							if (FlatIdent_67F21 == 0) then
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
		end
	end
	local function gString(Len)
		local FlatIdent_69270 = 0;
		local Str;
		local FStr;
		while true do
			if (FlatIdent_69270 == 3) then
				return Concat(FStr);
			end
			if (FlatIdent_69270 == 0) then
				Str = nil;
				if not Len then
					local FlatIdent_8D83D = 0;
					while true do
						if (FlatIdent_8D83D == 0) then
							Len = gBits32();
							if (Len == 0) then
								return "";
							end
							break;
						end
					end
				end
				FlatIdent_69270 = 1;
			end
			if (FlatIdent_69270 == 2) then
				FStr = {};
				for Idx = 1, #Str do
					FStr[Idx] = Char(Byte(Sub(Str, Idx, Idx)));
				end
				FlatIdent_69270 = 3;
			end
			if (FlatIdent_69270 == 1) then
				Str = Sub(ByteString, DIP, (DIP + Len) - 1);
				DIP = DIP + Len;
				FlatIdent_69270 = 2;
			end
		end
	end
	local gInt = gBits32;
	local function _R(...)
		return {...}, Select("#", ...);
	end
	local function Deserialize()
		local Instrs = {};
		local Functions = {};
		local Lines = {};
		local Chunk = {Instrs,Functions,nil,Lines};
		local ConstCount = gBits32();
		local Consts = {};
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
		for Idx = 1, gBits32() do
			local FlatIdent_1743D = 0;
			local Descriptor;
			while true do
				if (FlatIdent_1743D == 0) then
					Descriptor = gBits8();
					if (gBit(Descriptor, 1, 1) == 0) then
						local FlatIdent_7366E = 0;
						local Type;
						local Mask;
						local Inst;
						while true do
							if (FlatIdent_7366E == 3) then
								if (gBit(Mask, 3, 3) == 1) then
									Inst[4] = Consts[Inst[4]];
								end
								Instrs[Idx] = Inst;
								break;
							end
							if (FlatIdent_7366E == 1) then
								Inst = {gBits16(),gBits16(),nil,nil};
								if (Type == 0) then
									local FlatIdent_61585 = 0;
									while true do
										if (0 == FlatIdent_61585) then
											Inst[3] = gBits16();
											Inst[4] = gBits16();
											break;
										end
									end
								elseif (Type == 1) then
									Inst[3] = gBits32();
								elseif (Type == 2) then
									Inst[3] = gBits32() - (2 ^ 16);
								elseif (Type == 3) then
									Inst[3] = gBits32() - (2 ^ 16);
									Inst[4] = gBits16();
								end
								FlatIdent_7366E = 2;
							end
							if (0 == FlatIdent_7366E) then
								Type = gBit(Descriptor, 2, 3);
								Mask = gBit(Descriptor, 4, 6);
								FlatIdent_7366E = 1;
							end
							if (2 == FlatIdent_7366E) then
								if (gBit(Mask, 1, 1) == 1) then
									Inst[2] = Consts[Inst[2]];
								end
								if (gBit(Mask, 2, 2) == 1) then
									Inst[3] = Consts[Inst[3]];
								end
								FlatIdent_7366E = 3;
							end
						end
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
				local FlatIdent_7F35E = 0;
				while true do
					if (1 == FlatIdent_7F35E) then
						if (Enum <= 27) then
							if (Enum <= 13) then
								if (Enum <= 6) then
									if (Enum <= 2) then
										if (Enum <= 0) then
											Stk[Inst[2]] = Stk[Inst[3]][Stk[Inst[4]]];
										elseif (Enum == 1) then
											for Idx = Inst[2], Inst[3] do
												Stk[Idx] = nil;
											end
										else
											Stk[Inst[2]] = Stk[Inst[3]] + Stk[Inst[4]];
										end
									elseif (Enum <= 4) then
										if (Enum == 3) then
											Stk[Inst[2]] = Upvalues[Inst[3]];
										else
											local FlatIdent_1BAD7 = 0;
											local A;
											while true do
												if (6 == FlatIdent_1BAD7) then
													VIP = VIP + 1;
													Inst = Instr[VIP];
													VIP = Inst[3];
													break;
												end
												if (FlatIdent_1BAD7 == 3) then
													Inst = Instr[VIP];
													A = Inst[2];
													Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
													FlatIdent_1BAD7 = 4;
												end
												if (FlatIdent_1BAD7 == 1) then
													Inst = Instr[VIP];
													Stk[Inst[2]] = Stk[Inst[3]];
													VIP = VIP + 1;
													FlatIdent_1BAD7 = 2;
												end
												if (0 == FlatIdent_1BAD7) then
													A = nil;
													Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
													VIP = VIP + 1;
													FlatIdent_1BAD7 = 1;
												end
												if (FlatIdent_1BAD7 == 4) then
													VIP = VIP + 1;
													Inst = Instr[VIP];
													Stk[Inst[2]] = Stk[Inst[3]];
													FlatIdent_1BAD7 = 5;
												end
												if (FlatIdent_1BAD7 == 5) then
													VIP = VIP + 1;
													Inst = Instr[VIP];
													Stk[Inst[2]] = Inst[3];
													FlatIdent_1BAD7 = 6;
												end
												if (FlatIdent_1BAD7 == 2) then
													Inst = Instr[VIP];
													Stk[Inst[2]] = Inst[3];
													VIP = VIP + 1;
													FlatIdent_1BAD7 = 3;
												end
											end
										end
									elseif (Enum == 5) then
										Stk[Inst[2]][Stk[Inst[3]]] = Stk[Inst[4]];
									else
										Upvalues[Inst[3]] = Stk[Inst[2]];
									end
								elseif (Enum <= 9) then
									if (Enum <= 7) then
										local FlatIdent_455BF = 0;
										local A;
										while true do
											if (FlatIdent_455BF == 3) then
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Env[Inst[3]];
												FlatIdent_455BF = 4;
											end
											if (FlatIdent_455BF == 4) then
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
												FlatIdent_455BF = 5;
											end
											if (FlatIdent_455BF == 5) then
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
												FlatIdent_455BF = 6;
											end
											if (FlatIdent_455BF == 2) then
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]][Inst[3]] = Inst[4];
												FlatIdent_455BF = 3;
											end
											if (FlatIdent_455BF == 7) then
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]][Inst[3]] = Inst[4];
												FlatIdent_455BF = 8;
											end
											if (FlatIdent_455BF == 6) then
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
												FlatIdent_455BF = 7;
											end
											if (FlatIdent_455BF == 0) then
												A = nil;
												A = Inst[2];
												Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
												FlatIdent_455BF = 1;
											end
											if (FlatIdent_455BF == 8) then
												VIP = VIP + 1;
												Inst = Instr[VIP];
												for Idx = Inst[2], Inst[3] do
													Stk[Idx] = nil;
												end
												break;
											end
											if (FlatIdent_455BF == 1) then
												VIP = VIP + 1;
												Inst = Instr[VIP];
												Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
												FlatIdent_455BF = 2;
											end
										end
									elseif (Enum > 8) then
										local A;
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
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
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										A = Inst[2];
										Stk[A] = Stk[A](Stk[A + 1]);
										VIP = VIP + 1;
										Inst = Instr[VIP];
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
										Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
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
										Inst = Instr[VIP];
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
										VIP = VIP + 1;
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
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
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
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										A = Inst[2];
										Stk[A] = Stk[A](Stk[A + 1]);
										VIP = VIP + 1;
										Inst = Instr[VIP];
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
										Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
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
										Inst = Instr[VIP];
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
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
									else
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
									end
								elseif (Enum <= 11) then
									if (Enum > 10) then
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]][Inst[3]] = Inst[4];
									else
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Env[Inst[3]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										if (Stk[Inst[2]] ~= Stk[Inst[4]]) then
											VIP = VIP + 1;
										else
											VIP = Inst[3];
										end
									end
								elseif (Enum == 12) then
									if not Stk[Inst[2]] then
										VIP = VIP + 1;
									else
										VIP = Inst[3];
									end
								else
									local FlatIdent_74348 = 0;
									local B;
									local A;
									while true do
										if (FlatIdent_74348 == 4) then
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											A = Inst[2];
											FlatIdent_74348 = 5;
										end
										if (FlatIdent_74348 == 3) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											A = Inst[2];
											Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
											VIP = VIP + 1;
											FlatIdent_74348 = 4;
										end
										if (FlatIdent_74348 == 0) then
											B = nil;
											A = nil;
											A = Inst[2];
											B = Stk[Inst[3]];
											Stk[A + 1] = B;
											FlatIdent_74348 = 1;
										end
										if (FlatIdent_74348 == 2) then
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]];
											FlatIdent_74348 = 3;
										end
										if (FlatIdent_74348 == 6) then
											A = Inst[2];
											Stk[A](Stk[A + 1]);
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											break;
										end
										if (FlatIdent_74348 == 5) then
											B = Stk[Inst[3]];
											Stk[A + 1] = B;
											Stk[A] = B[Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_74348 = 6;
										end
										if (FlatIdent_74348 == 1) then
											Stk[A] = B[Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]];
											VIP = VIP + 1;
											FlatIdent_74348 = 2;
										end
									end
								end
							elseif (Enum <= 20) then
								if (Enum <= 16) then
									if (Enum <= 14) then
										Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
									elseif (Enum > 15) then
										local FlatIdent_270C = 0;
										local A;
										while true do
											if (FlatIdent_270C == 0) then
												A = Inst[2];
												Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
												break;
											end
										end
									elseif Stk[Inst[2]] then
										VIP = VIP + 1;
									else
										VIP = Inst[3];
									end
								elseif (Enum <= 18) then
									if (Enum == 17) then
										local A;
										Stk[Inst[2]] = Upvalues[Inst[3]];
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
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
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
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Env[Inst[3]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Env[Inst[3]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										A = Inst[2];
										Stk[A](Unpack(Stk, A + 1, Inst[3]));
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Env[Inst[3]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										A = Inst[2];
										Stk[A](Stk[A + 1]);
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
									else
										local B = Inst[3];
										local K = Stk[B];
										for Idx = B + 1, Inst[4] do
											K = K .. Stk[Idx];
										end
										Stk[Inst[2]] = K;
									end
								elseif (Enum > 19) then
									Stk[Inst[2]] = {};
								else
									local A;
									Stk[Inst[2]] = Upvalues[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]] - Stk[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Upvalues[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Upvalues[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Upvalues[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]] + Stk[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Upvalues[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Upvalues[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]] + Stk[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									A = Inst[2];
									Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									VIP = Inst[3];
								end
							elseif (Enum <= 23) then
								if (Enum <= 21) then
									do
										return;
									end
								elseif (Enum == 22) then
									local A;
									A = Inst[2];
									Stk[A](Unpack(Stk, A + 1, Inst[3]));
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = {};
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Inst[4];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Inst[4];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Inst[4];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Inst[4];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Inst[4];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Inst[4];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Inst[4];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Inst[4];
								else
									Stk[Inst[2]] = Env[Inst[3]];
								end
							elseif (Enum <= 25) then
								if (Enum == 24) then
									local FlatIdent_5CC3B = 0;
									local B;
									local A;
									while true do
										if (FlatIdent_5CC3B == 7) then
											Stk[A] = B[Inst[4]];
											break;
										end
										if (FlatIdent_5CC3B == 0) then
											B = nil;
											A = nil;
											A = Inst[2];
											Stk[A](Unpack(Stk, A + 1, Inst[3]));
											FlatIdent_5CC3B = 1;
										end
										if (FlatIdent_5CC3B == 2) then
											Inst = Instr[VIP];
											A = Inst[2];
											B = Stk[Inst[3]];
											Stk[A + 1] = B;
											FlatIdent_5CC3B = 3;
										end
										if (FlatIdent_5CC3B == 3) then
											Stk[A] = B[Inst[4]];
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											FlatIdent_5CC3B = 4;
										end
										if (FlatIdent_5CC3B == 5) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											VIP = VIP + 1;
											FlatIdent_5CC3B = 6;
										end
										if (FlatIdent_5CC3B == 6) then
											Inst = Instr[VIP];
											A = Inst[2];
											B = Stk[Inst[3]];
											Stk[A + 1] = B;
											FlatIdent_5CC3B = 7;
										end
										if (FlatIdent_5CC3B == 1) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Env[Inst[3]];
											VIP = VIP + 1;
											FlatIdent_5CC3B = 2;
										end
										if (FlatIdent_5CC3B == 4) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											A = Inst[2];
											Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
											FlatIdent_5CC3B = 5;
										end
									end
								else
									Stk[Inst[2]] = Upvalues[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									if (Stk[Inst[2]] == Stk[Inst[4]]) then
										VIP = VIP + 1;
									else
										VIP = Inst[3];
									end
								end
							elseif (Enum > 26) then
								VIP = Inst[3];
							else
								local FlatIdent_30B1F = 0;
								local Edx;
								local Results;
								local A;
								while true do
									if (FlatIdent_30B1F == 5) then
										Inst = Instr[VIP];
										A = Inst[2];
										Results = {Stk[A](Stk[A + 1])};
										FlatIdent_30B1F = 6;
									end
									if (FlatIdent_30B1F == 3) then
										Inst = Instr[VIP];
										Stk[Inst[2]] = Env[Inst[3]];
										VIP = VIP + 1;
										FlatIdent_30B1F = 4;
									end
									if (FlatIdent_30B1F == 6) then
										Edx = 0;
										for Idx = A, Inst[4] do
											local FlatIdent_8D1A5 = 0;
											while true do
												if (0 == FlatIdent_8D1A5) then
													Edx = Edx + 1;
													Stk[Idx] = Results[Edx];
													break;
												end
											end
										end
										VIP = VIP + 1;
										FlatIdent_30B1F = 7;
									end
									if (FlatIdent_30B1F == 1) then
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_30B1F = 2;
									end
									if (FlatIdent_30B1F == 0) then
										Edx = nil;
										Results = nil;
										A = nil;
										FlatIdent_30B1F = 1;
									end
									if (FlatIdent_30B1F == 2) then
										A = Inst[2];
										Stk[A](Stk[A + 1]);
										VIP = VIP + 1;
										FlatIdent_30B1F = 3;
									end
									if (FlatIdent_30B1F == 7) then
										Inst = Instr[VIP];
										VIP = Inst[3];
										break;
									end
									if (4 == FlatIdent_30B1F) then
										Inst = Instr[VIP];
										Stk[Inst[2]] = Upvalues[Inst[3]];
										VIP = VIP + 1;
										FlatIdent_30B1F = 5;
									end
								end
							end
						elseif (Enum <= 41) then
							if (Enum <= 34) then
								if (Enum <= 30) then
									if (Enum <= 28) then
										local FlatIdent_7DCBC = 0;
										local A;
										local Cls;
										while true do
											if (FlatIdent_7DCBC == 0) then
												A = Inst[2];
												Cls = {};
												FlatIdent_7DCBC = 1;
											end
											if (FlatIdent_7DCBC == 1) then
												for Idx = 1, #Lupvals do
													local FlatIdent_47A85 = 0;
													local List;
													while true do
														if (FlatIdent_47A85 == 0) then
															List = Lupvals[Idx];
															for Idz = 0, #List do
																local Upv = List[Idz];
																local NStk = Upv[1];
																local DIP = Upv[2];
																if ((NStk == Stk) and (DIP >= A)) then
																	local FlatIdent_1EAB2 = 0;
																	while true do
																		if (0 == FlatIdent_1EAB2) then
																			Cls[DIP] = NStk[DIP];
																			Upv[1] = Cls;
																			break;
																		end
																	end
																end
															end
															break;
														end
													end
												end
												break;
											end
										end
									elseif (Enum > 29) then
										local A = Inst[2];
										local C = Inst[4];
										local CB = A + 2;
										local Result = {Stk[A](Stk[A + 1], Stk[CB])};
										for Idx = 1, C do
											Stk[CB + Idx] = Result[Idx];
										end
										local R = Result[1];
										if R then
											local FlatIdent_8B523 = 0;
											while true do
												if (FlatIdent_8B523 == 0) then
													Stk[CB] = R;
													VIP = Inst[3];
													break;
												end
											end
										else
											VIP = VIP + 1;
										end
									else
										Stk[Inst[2]] = Stk[Inst[3]] - Stk[Inst[4]];
									end
								elseif (Enum <= 32) then
									if (Enum > 31) then
										local FlatIdent_86634 = 0;
										local NewProto;
										local NewUvals;
										local Indexes;
										while true do
											if (FlatIdent_86634 == 2) then
												for Idx = 1, Inst[4] do
													VIP = VIP + 1;
													local Mvm = Instr[VIP];
													if (Mvm[1] == 49) then
														Indexes[Idx - 1] = {Stk,Mvm[3]};
													else
														Indexes[Idx - 1] = {Upvalues,Mvm[3]};
													end
													Lupvals[#Lupvals + 1] = Indexes;
												end
												Stk[Inst[2]] = Wrap(NewProto, NewUvals, Env);
												break;
											end
											if (FlatIdent_86634 == 1) then
												Indexes = {};
												NewUvals = Setmetatable({}, {__index=function(_, Key)
													local FlatIdent_20FB0 = 0;
													local Val;
													while true do
														if (0 == FlatIdent_20FB0) then
															Val = Indexes[Key];
															return Val[1][Val[2]];
														end
													end
												end,__newindex=function(_, Key, Value)
													local Val = Indexes[Key];
													Val[1][Val[2]] = Value;
												end});
												FlatIdent_86634 = 2;
											end
											if (FlatIdent_86634 == 0) then
												NewProto = Proto[Inst[3]];
												NewUvals = nil;
												FlatIdent_86634 = 1;
											end
										end
									else
										local FlatIdent_5B4A8 = 0;
										local B;
										local A;
										while true do
											if (FlatIdent_5B4A8 == 19) then
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
												FlatIdent_5B4A8 = 20;
											end
											if (0 == FlatIdent_5B4A8) then
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
												FlatIdent_5B4A8 = 1;
											end
											if (9 == FlatIdent_5B4A8) then
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
												FlatIdent_5B4A8 = 10;
											end
											if (16 == FlatIdent_5B4A8) then
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
												FlatIdent_5B4A8 = 17;
											end
											if (FlatIdent_5B4A8 == 3) then
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
												FlatIdent_5B4A8 = 4;
											end
											if (FlatIdent_5B4A8 == 20) then
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
												FlatIdent_5B4A8 = 21;
											end
											if (FlatIdent_5B4A8 == 6) then
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
												FlatIdent_5B4A8 = 7;
											end
											if (FlatIdent_5B4A8 == 2) then
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
												FlatIdent_5B4A8 = 3;
											end
											if (FlatIdent_5B4A8 == 12) then
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
												FlatIdent_5B4A8 = 13;
											end
											if (FlatIdent_5B4A8 == 11) then
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
												FlatIdent_5B4A8 = 12;
											end
											if (FlatIdent_5B4A8 == 18) then
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
												FlatIdent_5B4A8 = 19;
											end
											if (FlatIdent_5B4A8 == 25) then
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
											if (FlatIdent_5B4A8 == 15) then
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
												FlatIdent_5B4A8 = 16;
											end
											if (7 == FlatIdent_5B4A8) then
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
												FlatIdent_5B4A8 = 8;
											end
											if (FlatIdent_5B4A8 == 13) then
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
												FlatIdent_5B4A8 = 14;
											end
											if (FlatIdent_5B4A8 == 24) then
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
												FlatIdent_5B4A8 = 25;
											end
											if (FlatIdent_5B4A8 == 1) then
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
												FlatIdent_5B4A8 = 2;
											end
											if (FlatIdent_5B4A8 == 23) then
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
												FlatIdent_5B4A8 = 24;
											end
											if (FlatIdent_5B4A8 == 21) then
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
												FlatIdent_5B4A8 = 22;
											end
											if (FlatIdent_5B4A8 == 5) then
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
												FlatIdent_5B4A8 = 6;
											end
											if (FlatIdent_5B4A8 == 8) then
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
												FlatIdent_5B4A8 = 9;
											end
											if (FlatIdent_5B4A8 == 22) then
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
												FlatIdent_5B4A8 = 23;
											end
											if (FlatIdent_5B4A8 == 4) then
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
												FlatIdent_5B4A8 = 5;
											end
											if (FlatIdent_5B4A8 == 10) then
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
												FlatIdent_5B4A8 = 11;
											end
											if (FlatIdent_5B4A8 == 17) then
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
												FlatIdent_5B4A8 = 18;
											end
											if (FlatIdent_5B4A8 == 14) then
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
												FlatIdent_5B4A8 = 15;
											end
										end
									end
								elseif (Enum > 33) then
									local FlatIdent_91B54 = 0;
									local A;
									while true do
										if (FlatIdent_91B54 == 0) then
											A = Inst[2];
											Stk[A](Stk[A + 1]);
											break;
										end
									end
								else
									local B;
									local A;
									Stk[Inst[2]] = Stk[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]]();
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
									Stk[Inst[2]][Inst[3]] = Inst[4];
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
									Inst = Instr[VIP];
									A = Inst[2];
									Stk[A] = Stk[A](Stk[A + 1]);
									VIP = VIP + 1;
									Inst = Instr[VIP];
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
									VIP = VIP + 1;
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
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									A = Inst[2];
									Stk[A] = Stk[A](Stk[A + 1]);
									VIP = VIP + 1;
									Inst = Instr[VIP];
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
									Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Inst[4];
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
									Inst = Instr[VIP];
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
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Inst[4];
								end
							elseif (Enum <= 37) then
								if (Enum <= 35) then
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
								elseif (Enum > 36) then
									if (Stk[Inst[2]] == Inst[4]) then
										VIP = VIP + 1;
									else
										VIP = Inst[3];
									end
								else
									local FlatIdent_6679B = 0;
									local B;
									local A;
									while true do
										if (FlatIdent_6679B == 1) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											FlatIdent_6679B = 2;
										end
										if (FlatIdent_6679B == 0) then
											B = nil;
											A = nil;
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											FlatIdent_6679B = 1;
										end
										if (FlatIdent_6679B == 5) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]];
											FlatIdent_6679B = 6;
										end
										if (FlatIdent_6679B == 4) then
											B = Stk[Inst[3]];
											Stk[A + 1] = B;
											Stk[A] = B[Inst[4]];
											FlatIdent_6679B = 5;
										end
										if (FlatIdent_6679B == 3) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											A = Inst[2];
											FlatIdent_6679B = 4;
										end
										if (FlatIdent_6679B == 2) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
											FlatIdent_6679B = 3;
										end
										if (FlatIdent_6679B == 8) then
											Stk[A](Unpack(Stk, A + 1, Inst[3]));
											VIP = VIP + 1;
											Inst = Instr[VIP];
											FlatIdent_6679B = 9;
										end
										if (FlatIdent_6679B == 6) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											Stk[Inst[2]] = Inst[3];
											FlatIdent_6679B = 7;
										end
										if (FlatIdent_6679B == 7) then
											VIP = VIP + 1;
											Inst = Instr[VIP];
											A = Inst[2];
											FlatIdent_6679B = 8;
										end
										if (FlatIdent_6679B == 9) then
											VIP = Inst[3];
											break;
										end
									end
								end
							elseif (Enum <= 39) then
								if (Enum == 38) then
									local A;
									Stk[Inst[2]] = Upvalues[Inst[3]];
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
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
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
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									A = Inst[2];
									Stk[A](Unpack(Stk, A + 1, Inst[3]));
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Env[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									A = Inst[2];
									Stk[A](Stk[A + 1]);
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									VIP = Inst[3];
								else
									local FlatIdent_1F138 = 0;
									local A;
									while true do
										if (FlatIdent_1F138 == 0) then
											A = Inst[2];
											Stk[A] = Stk[A]();
											break;
										end
									end
								end
							elseif (Enum > 40) then
								local FlatIdent_61AEE = 0;
								while true do
									if (3 == FlatIdent_61AEE) then
										VIP = Inst[3];
										break;
									end
									if (FlatIdent_61AEE == 1) then
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Upvalues[Inst[3]] = Stk[Inst[2]];
										VIP = VIP + 1;
										FlatIdent_61AEE = 2;
									end
									if (FlatIdent_61AEE == 2) then
										Inst = Instr[VIP];
										Stk[Inst[2]] = Inst[3];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										FlatIdent_61AEE = 3;
									end
									if (FlatIdent_61AEE == 0) then
										Upvalues[Inst[3]] = Stk[Inst[2]];
										VIP = VIP + 1;
										Inst = Instr[VIP];
										Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
										FlatIdent_61AEE = 1;
									end
								end
							else
								local FlatIdent_7063 = 0;
								local A;
								local Results;
								local Edx;
								while true do
									if (FlatIdent_7063 == 0) then
										A = Inst[2];
										Results = {Stk[A](Stk[A + 1])};
										FlatIdent_7063 = 1;
									end
									if (FlatIdent_7063 == 1) then
										Edx = 0;
										for Idx = A, Inst[4] do
											Edx = Edx + 1;
											Stk[Idx] = Results[Edx];
										end
										break;
									end
								end
							end
						elseif (Enum <= 48) then
							if (Enum <= 44) then
								if (Enum <= 42) then
									local A = Inst[2];
									Stk[A](Unpack(Stk, A + 1, Inst[3]));
								elseif (Enum == 43) then
									local A;
									Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									A = Inst[2];
									Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = {};
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]][Inst[3]] = Stk[Inst[4]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Stk[Inst[3]];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									Stk[Inst[2]] = Inst[3];
									VIP = VIP + 1;
									Inst = Instr[VIP];
									VIP = Inst[3];
								else
									Stk[Inst[2]]();
								end
							elseif (Enum <= 46) then
								if (Enum == 45) then
									if (Stk[Inst[2]] ~= Stk[Inst[4]]) then
										VIP = VIP + 1;
									else
										VIP = Inst[3];
									end
								else
									Stk[Inst[2]] = Inst[3] ~= 0;
								end
							elseif (Enum == 47) then
								Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
							else
								local A = Inst[2];
								Stk[A] = Stk[A](Stk[A + 1]);
							end
						elseif (Enum <= 52) then
							if (Enum <= 50) then
								if (Enum > 49) then
									Stk[Inst[2]][Inst[3]] = Inst[4];
								else
									Stk[Inst[2]] = Stk[Inst[3]];
								end
							elseif (Enum == 51) then
								Stk[Inst[2]][Inst[3]] = Inst[4];
								VIP = VIP + 1;
								Inst = Instr[VIP];
								Stk[Inst[2]][Inst[3]] = Inst[4];
								VIP = VIP + 1;
								Inst = Instr[VIP];
								Stk[Inst[2]][Inst[3]] = Inst[4];
								VIP = VIP + 1;
								Inst = Instr[VIP];
								Stk[Inst[2]][Inst[3]] = Inst[4];
								VIP = VIP + 1;
								Inst = Instr[VIP];
								Stk[Inst[2]][Inst[3]] = Inst[4];
							else
								local B;
								local A;
								Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
								VIP = VIP + 1;
								Inst = Instr[VIP];
								A = Inst[2];
								B = Stk[Inst[3]];
								Stk[A + 1] = B;
								Stk[A] = B[Inst[4]];
								VIP = VIP + 1;
								Inst = Instr[VIP];
								A = Inst[2];
								Stk[A] = Stk[A](Stk[A + 1]);
								VIP = VIP + 1;
								Inst = Instr[VIP];
								Stk[Inst[2]] = Stk[Inst[3]];
								VIP = VIP + 1;
								Inst = Instr[VIP];
								A = Inst[2];
								B = Stk[Inst[3]];
								Stk[A + 1] = B;
								Stk[A] = B[Inst[4]];
								VIP = VIP + 1;
								Inst = Instr[VIP];
								Stk[Inst[2]] = Inst[3];
								VIP = VIP + 1;
								Inst = Instr[VIP];
								A = Inst[2];
								Stk[A] = Stk[A](Unpack(Stk, A + 1, Inst[3]));
								VIP = VIP + 1;
								Inst = Instr[VIP];
								Stk[Inst[2]] = Stk[Inst[3]];
								VIP = VIP + 1;
								Inst = Instr[VIP];
								Stk[Inst[2]] = Inst[3];
							end
						elseif (Enum <= 54) then
							if (Enum > 53) then
								local FlatIdent_3B08E = 0;
								local A;
								local B;
								while true do
									if (FlatIdent_3B08E == 0) then
										A = Inst[2];
										B = Stk[Inst[3]];
										FlatIdent_3B08E = 1;
									end
									if (FlatIdent_3B08E == 1) then
										Stk[A + 1] = B;
										Stk[A] = B[Inst[4]];
										break;
									end
								end
							else
								Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
								VIP = VIP + 1;
								Inst = Instr[VIP];
								Stk[Inst[2]] = Env[Inst[3]];
								VIP = VIP + 1;
								Inst = Instr[VIP];
								Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
								VIP = VIP + 1;
								Inst = Instr[VIP];
								Stk[Inst[2]] = Stk[Inst[3]][Inst[4]];
								VIP = VIP + 1;
								Inst = Instr[VIP];
								if (Stk[Inst[2]] ~= Stk[Inst[4]]) then
									VIP = VIP + 1;
								else
									VIP = Inst[3];
								end
							end
						elseif (Enum == 55) then
							Stk[Inst[2]] = Inst[3];
						elseif (Stk[Inst[2]] == Stk[Inst[4]]) then
							VIP = VIP + 1;
						else
							VIP = Inst[3];
						end
						VIP = VIP + 1;
						break;
					end
					if (FlatIdent_7F35E == 0) then
						Inst = Instr[VIP];
						Enum = Inst[1];
						FlatIdent_7F35E = 1;
					end
				end
			end
		end;
	end
	return Wrap(Deserialize(), {}, vmenv)(...);
end
return VMCall("LOL!8C3O0003073O0067657467656E7603103O004E6F74696669636174696F6E5465787403283O0054797065202F636F6E736F6C6520746F20732O6520746865206C697374206F6620776F7264203B2903083O00496E7374616E63652O033O006E657703093O005363722O656E47756903063O00506172656E7403043O0067616D6503073O00506C6179657273030B3O004C6F63616C506C61796572030C3O0057616974466F724368696C6403093O00506C6179657247756903053O004672616D6503043O0053697A6503053O005544696D32028O00025O00C07240026O00594003083O00506F736974696F6E026O00E03F025O00C062C0026O0059C003103O004261636B67726F756E64436F6C6F723303063O00436F6C6F723303073O0066726F6D524742030F3O00426F7264657253697A65506978656C03093O00546578744C6162656C026O00F03F03163O004261636B67726F756E645472616E73706172656E6379030A3O0054657874436F6C6F7233025O00E06F40030A3O00546578745363616C65642O0103043O0054657874030A3O0047657453657276696365030C3O0054772O656E5365727669636503043O004E616D6503353O004E657074756E6520427970612O736572206D616E75616C20286D61646520627920646162752073686F776361736520696E20797429026O007940026O006940026O0069C0026O004940033A3O004E657074756E6520627970612O736572206D616E75616C20286D61646520627920646162752073686F776361736520646F6E277420736B696429026O003E4003043O00466F6E7403043O00456E756D030A3O00476F7468616D426F6C6403083O005465787453697A65026O00344003073O0054657874426F78029A5O99E93F026O33D33F029A5O99B93F026O66D63F030F3O00506C616365686F6C64657254657874030F3O00456E746572206120776F72643O2E03063O00476F7468616D026O003240030A3O005465787442752O746F6E029A5O99C93F026O66E63F030B3O004175746F636F2O72656374025O00805140030A3O00496E707574426567616E03073O00436F2O6E656374030C3O00496E7075744368616E67656403103O0055736572496E7075745365727669636503043O006675636B03133O00D293CCB2E1B49CCCB2E1B484CCB2E1B48BCCB203043O00636F636B03143O00E1B484CCB2E1B48FCCB2E1B484CCB2E1B48BCCB203053O006E692O676103153O00C9B4CCB2C9AACCB2C9A2CCB2C9A2CCB2E1B480CCB203063O0066656D626F79031B3O00D293CCB2E1B487CCB2E1B48DCCB2CA99CCB2E1B48FCCB2CA8FCCB22O033O00676179030D3O00C9A2CCB2E1B480CCB2CA8FCCB203043O00706F726E03123O00E1B498CCB2E1B48FCCB2CA80CCB2C9B4CCB203093O007065646F7068696C65032A3O00E1B498CCB2E1B487CCB2E1B485CCB2E1B48FCCB2E1B498CCB2CA9CCCB2C9AACCB2CA9FCCB2E1B487CCB203043O007368697403103O0073CCB2CA9CCCB2C9AACCB2E1B49BCCB22O033O0063756D030F3O00E1B484CCB2E1B49CCCB2E1B48DCCB203053O00626974636803163O00CA99CCB2C9AACCB2E1B49BCCB2E1B484CCB2CA9CCCB203053O0070752O737903143O00E1B498CCB2E1B49CCCB273CCB273CCB2CA8FCCB22O033O00736578033D3O0073CCB3D0B5CCB3E281A5CCB3E281A5CCB3E281A5CCB3E281A5CCB3E281A5CCB3E281A5CCB3E281A5CCB3E281A5CCB3E281A5CCB3E281A5CCB3D185CCB303043O00736C757403113O0073CC85CA9FCC85E1B49CCC85E1B49BCC8503043O0063756E7403133O00E1B484CCB2E1B49CCCB2C9B4CCB2E1B49BCCB203043O007469747303113O00E1B49BCCB2C9AACCB2E1B49BCCB273CCB22O033O006B7973030C3O00E1B48BCCB2CA8FCCB273CCB203043O006469636B03133O00E1B485CCB2C9AACCB2E1B484CCB2E1B48BCCB203053O00622O6F627303153O00CA99CCB2E1B48FCCB2E1B48FCCB2CA99CCB273CCB22O033O00612O73030B3O00E1B480CCB273CCB273CCB203073O006675636B696E67031F3O00D293CCB2E1B49CCCB2E1B484CCB2E1B48BCCB2C9AACCB2C9B4CCB2C9A2CCB203063O006D696E6F727303173O004DCCB2C9AACCB2C9B4CCB2E1B48FCCB2CA80CCB273CCB22O033O00686F65030E3O00CA9CCCB2E1B48FCCB2E1B487CCB203053O00686F726E7903153O00CA9CCCB2E1B48FCCB2CA80CCB2C9B4CCB2CA8FCCB203063O00726574617264031C3O00CA80CCB2E1B487CCB2E1B49BCCB2E1B480CCB2CA80CCB2E1B485CCB203063O0066612O676F74031B3O00D293CCB2E1B480CCB2C9A2CCB2C9A2CCB2E1B48FCCB2E1B49BCCB2030A3O006D617374757262617465032E3O00E1B48DCCB2E1B480CCB273CCB2E1B49BCCB2E1B49CCCB2CA80CCB2CA99CCB2E1B480CCB2E1B49BCCB2E1B487CCB203053O00736C61766503163O0073CCB2CA9FCCB2E1B480CCB2E1B4A0CCB2E1B487CCB203063O00736C6176657303193O0073CCB2CA9FCCB2E1B480CCB2E1B4A0CCB2E1B487CCB273CCB203053O006E616B656403183O00C9B4CCB2E1B480CCB2E1B48BCCB2E1B487CCB2E1B485CCB203073O006675636B657273031F3O00D293CCB2E1B49CCCB2E1B484CCB2E1B48BCCB2E1B487CCB2CA80CCB273CCB203063O00736C752O7479031A3O0073CC85CA9FCC85E1B49CCC85E1B49BCC85E1B49BCC85CA8FCC8503063O0074692O746573031B3O00E1B49BCCB2C9AACCB2E1B49BCCB2E1B49BCCB2E1B487CCB273CCB203063O006E692O67657203193O00C9B4CCB2C9AACCB2C9A2CCB2C9A2CCB2E1B487CCB2CA80CCB203073O006C65736269616E031D3O00CA9FCCB2E1B487CCB273CCB2CA99CCB2C9AACCB2E1B480CCB2C9B4CCB203053O004C4742545103183O00CA9FCCB2C9A2CCB2CA99CCB2E1B49BCCB2E1B48FCCB2CCA803053O006C67622O7103173O00CA9FCCB2C9A2CCB2CA99CCB2CA99CCB2E1B48FCCB2CCA803113O004D6F75736542752O746F6E31436C69636B004D012O00121F3O00018O0001000200304O0002000300124O00043O00206O000500122O000100068O0002000200122O000100083O00202O00010001000900202O00010001000A00202O00010001000B00122O0003000C6O00010003000200104O0007000100122O000100043O00202O00010001000500122O0002000D6O00010002000200122O0002000F3O00202O00020002000500122O000300103O00122O000400113O00122O000500103O00122O000600126O00020006000200102O0001000E000200122O0002000F3O00202O00020002000500122O000300143O00122O000400153O00122O000500103O00122O000600166O00020006000200102O00010013000200122O000200183O00202O00020002001900122O000300103O00122O000400103O00122O000500106O00020005000200102O00010017000200302O0001001A001000102O000100073O00122O000200043O00202O00020002000500122O0003001B6O00020002000200122O0003000F3O00202O00030003000500122O0004001C3O00122O000500103O00122O0006001C3O00122O000700106O00030007000200102O0002000E000300122O0003000F3O00202O00030003000500122O000400103O00122O000500103O00122O000600103O00122O000700106O00030007000200102O00020013000300302O0002001D001C00122O000300183O00202O00030003001900122O0004001F3O00122O0005001F3O00122O0006001F6O00030006000200102O0002001E000300302O00020020002100122O000300016O00030001000200202O00030003000200102O00020022000300102O00020007000100122O000300083O00202O00030003002300122O000500244O001000030005000200062000043O000100012O00313O00033O00062000050001000100022O00313O00014O00313O00044O0021000600056O00060001000100122O000600043O00202O00060006000500122O000700066O00060002000200122O000700083O00202O00070007000900202O00070007000A00202O00070007000B00122O0009000C6O00070009000200102O00060007000700302O00060025002600122O000700043O00202O00070007000500122O0008000D6O00070002000200102O00070007000600122O0008000F3O00202O00080008000500122O000900103O00122O000A00273O00122O000B00103O00122O000C00286O0008000C000200102O0007000E000800122O0008000F3O00202O00080008000500122O000900143O00122O000A00293O00122O000B00143O00122O000C00166O0008000C000200102O00070013000800122O000800183O00202O00080008001900122O0009002A3O00122O000A002A3O00122O000B002A6O0008000B000200102O00070017000800302O0007001A001000122O000800043O00202O00080008000500122O0009001B6O00080002000200102O00080007000700122O0009000F3O00202O00090009000500122O000A001C3O00122O000B00103O00122O000C00103O00122O000D002A6O0009000D000200102O0008000E000900122O0009000F3O00202O00090009000500122O000A00103O00122O000B00103O00122O000C00103O00122O000D00106O0009000D000200102O00080013000900302O00080022002B00122O000900183O00202O00090009001900122O000A001F3O00122O000B001F3O00122O000C001F6O0009000C000200102O0008001E000900122O000900183O00202O00090009001900122O000A002C3O00122O000B002C3O00122O000C002C6O0009000C000200102O00080017000900302O0008001A00100012170009002E3O00200900090009002D00202O00090009002F00102O0008002D000900302O00080030003100122O000900043O00202O00090009000500122O000A00326O00090002000200102O00090007000700122O000A000F3O00202O000A000A000500122O000B00333O00122O000C00103O00122O000D00343O00122O000E00106O000A000E000200102O0009000E000A00122O000A000F3O00202O000A000A000500122O000B00353O00122O000C00103O00122O000D00363O00122O000E00106O000A000E000200102O00090013000A00302O00090037003800122O000A00183O00202O000A000A001900122O000B00103O00122O000C00103O00122O000D00106O000A000D000200102O0009001E000A00122O000A00183O00202O000A000A001900122O000B001F3O00122O000C001F3O00122O000D001F6O000A000D000200102O00090017000A00302O0009001A001000122O000A002E3O00202O000A000A002D00202O000A000A003900102O0009002D000A00302O00090030003A00122O000A00043O00202O000A000A000500122O000B003B6O000A0002000200102O000A0007000700122O000B000F3O00202O000B000B000500122O000C00333O00122O000D00103O00122O000E003C3O00122O000F00106O000B000F000200102O000A000E000B00122O000B000F3O00202O000B000B000500122O000C00353O00122O000D00103O00122O000E003D3O00122O000F00106O000B000F000200102O000A0013000B00302O000A0022003E00122O000B00183O00202O000B000B001900122O000C001F3O00122O000D001F3O00122O000E001F6O000B000E000200102O000A001E000B00122O000B00183O00202O000B000B001900122O000C003F3O00122O000D003F3O00122O000E003F4O0007000B000E000200102O000A0017000B00302O000A001A001000122O000B002E3O00202O000B000B002D00202O000B000B002F00102O000A002D000B00302O000A0030003A4O000B000E3O000620000F0002000100032O00313O000D4O00313O00074O00313O000E3O00202F00100007004000203600100010004100062000120003000100042O00313O000E4O00313O00074O00313O000B4O00313O000D4O002A00100012000100202F00100007004200203600100010004100062000120004000100012O00313O000C4O001800100012000100122O001000083O00202O00100010002300122O001200436O00100012000200202O00100010004200202O00100010004100062000120005000100032O00313O000C4O00313O000B4O00313O000F4O00160010001200014O00103O001900302O00100044004500302O00100046004700302O00100048004900302O0010004A004B00302O0010004C004D00302O0010004E004F00302O00100050005100302O00100052005300300B00100054005500302O00100056005700302O00100058005900302O0010005A005B00302O0010005C005D00302O0010005E005F00302O00100060006100302O00100062006300302O00100064006500302O00100066006700300B00100068006900302O0010006A006B00302O0010006C006D00302O0010006E006F00302O00100070007100302O00100072007300302O00100074007500302O00100076007700302O00100078007900302O0010007A007B0030330010007C007D00302O0010007E007F00302O00100080008100302O00100082008300302O0010008400850030320010008600870030320010008800890030320010008A008B00062000110006000100012O00313O00104O0031001200114O002C00120001000100202F0012000A008C00203600120012004100062000140007000100022O00313O00104O00313O00094O002A0012001400012O001C8O00153O00013O00083O000A3O00028O00026O00F03F03093O0054772O656E496E666F2O033O006E657703083O00506F736974696F6E03063O0043726561746503043O00506C6179027O004003093O00436F6D706C6574656403043O005761697405333O001237000500014O0001000600083O002625000500170001000100041B3O00170001001237000900013O002625000900090001000200041B3O00090001001237000500023O00041B3O00170001002625000900050001000100041B3O00050001001217000A00033O00202B000A000A00044O000B00026O000C00036O000D00046O000A000D00024O0006000A6O000A3O000100102O000A000500014O0007000A3O00122O000900023O00044O000500010026250005002B0001000200041B3O002B0001001237000900013O002625000900260001000100041B3O002600012O0003000A5O00200D000A000A00064O000C8O000D00066O000E00076O000A000E00024O0008000A3O00202O000A000800074O000A0002000100122O000900023O0026250009001A0001000200041B3O001A0001001237000500083O00041B3O002B000100041B3O001A0001002625000500020001000800041B3O0002000100202F00090008000900203600090009000A2O002200090002000100041B3O0032000100041B3O000200012O00153O00017O00113O00028O00027O004003073O0044657374726F7903053O005544696D322O033O006E6577026O00E03F025O00C062C0026O0049C003043O00456E756D030B3O00456173696E675374796C6503043O0053696E65030F3O00456173696E67446972656374696F6E2O033O004F757403043O0077616974026O00F03F026O00594003023O00496E003D3O0012373O00014O0001000100013O0026253O00020001000100041B3O00020001001237000100013O0026250001000B0001000200041B3O000B00012O000300025O0020360002000200032O002200020002000100041B3O003C0001002625000100220001000100041B3O002200012O0003000200014O001100035O00122O000400043O00202O00040004000500122O000500063O00122O000600073O00122O000700063O00122O000800086O00040008000200122O000500063O00122O000600093O00202O00060006000A00202O00060006000B00122O000700093O00202O00070007000C00202O00070007000D4O00020007000100122O0002000E3O00122O000300026O00020002000100122O0001000F3O002625000100050001000F00041B3O000500012O0003000200014O001100035O00122O000400043O00202O00040004000500122O000500063O00122O000600073O00122O0007000F3O00122O000800106O00040008000200122O000500063O00122O000600093O00202O00060006000A00202O00060006000B00122O000700093O00202O00070007000C00202O0007000700114O00020007000100122O0002000E3O00122O000300066O00020002000100122O000100023O00041B3O0005000100041B3O003C000100041B3O000200012O00153O00017O00083O00028O0003083O00506F736974696F6E03053O005544696D322O033O006E657703013O005803053O005363616C6503063O004F2O6673657403013O0059011F3O001237000100014O0001000200023O002625000100020001000100041B3O0002000100202F00033O00022O001300048O0002000300044O000300013O00122O000400033O00202O0004000400044O000500023O00202O00050005000500202O0005000500064O000600023O00202O00060006000500202O00060006000700202O0007000200054O0006000600074O000700023O00202O00070007000800202O0007000700064O000800023O00202O00080008000800202O00080008000700202O0009000200084O0008000800094O00040008000200102O00030002000400044O001E000100041B3O000200012O00153O00017O00093O00030D3O0055736572496E7075745479706503043O00456E756D030C3O004D6F75736542752O746F6E3103053O00546F756368028O00026O00F03F03083O00506F736974696F6E03073O004368616E67656403073O00436F2O6E65637401283O00200A00013O000100122O000200023O00202O00020002000100202O00020002000300062O0001000C0001000200041B3O000C000100202F00013O0001001217000200023O00202F00020002000100202F000200020004000638000100270001000200041B3O00270001001237000100054O0001000200023O0026250001000E0001000500041B3O000E0001001237000200053O0026250002001D0001000600041B3O001D00012O0003000300013O00202F0003000300072O000600035O00202F00033O000800203600030003000900062000053O000100022O00318O00033O00024O002A00030005000100041B3O00270001002625000200110001000500041B3O001100012O002E000300014O0029000300023O00202O00033O00074O000300033O00122O000200063O00044O0011000100041B3O0027000100041B3O000E00012O00153O00013O00013O00033O00030E3O0055736572496E707574537461746503043O00456E756D2O033O00456E64000A4O00197O00206O000100122O000100023O00202O00010001000100202O00010001000300064O00090001000100041B3O000900012O002E8O00063O00014O00153O00017O00043O00030D3O0055736572496E7075745479706503043O00456E756D030D3O004D6F7573654D6F76656D656E7403053O00546F756368010E3O00200A00013O000100122O000200023O00202O00020002000100202O00020002000300062O0001000C0001000200041B3O000C000100202F00013O0001001217000200023O00202F00020002000100202F0002000200040006380001000D0001000200041B3O000D00012O00068O00153O00019O002O00010A4O000300015O0006383O00090001000100041B3O000900012O0003000100013O00060F0001000900013O00041B3O000900012O0003000100024O003100026O00220001000200012O00153O00017O00053O00028O0003053O007072696E7403113O004175746F636F2O72656374204C6973743A03053O00706169727303043O00202D3E2000153O0012373O00013O0026253O00010001000100041B3O00010001001217000100023O00121A000200036O00010002000100122O000100046O00028O00010002000300044O00100001001217000600024O0023000700043O00122O000800056O000900056O0007000700094O00060002000100061E0001000A0001000200041B3O000A000100041B3O0014000100041B3O000100012O00153O00017O00103O00028O00027O004003043O0067616D6503113O005265706C69636174656453746F72616765031B3O0044656661756C744368617453797374656D436861744576656E747303113O005361794D652O7361676552657175657374030A3O00466972655365727665722O033O00412O6C026O00F03F03063O0069706169727303053O007461626C6503063O00636F6E63617403013O002003043O005465787403053O006C6F77657203053O0073706C697400403O0012373O00014O0001000100033O0026253O000D0001000200041B3O000D0001001217000400033O00202400040004000400202O00040004000500202O00040004000600202O0004000400074O000600033O00122O000700086O00040007000100044O003F00010026253O002A0001000900041B3O002A0001001237000400013O002625000400140001000900041B3O001400010012373O00023O00041B3O002A0001002625000400100001000100041B3O001000010012170005000A4O0031000600024O002800050002000700041B3O002000012O0003000A8O000A000A000900060C000A001F0001000100041B3O001F00012O0031000A00094O000500020008000A00061E0005001A0001000200041B3O001A00010012170005000B3O00200400050005000C4O000600023O00122O0007000D6O0005000700024O000300053O00122O000400093O00044O001000010026253O00020001000100041B3O00020001001237000400013O002625000400390001000100041B3O003900012O0003000500013O00203400050005000E00202O00050005000F4O0005000200024O000100053O00202O00050001001000122O0007000D6O0005000700024O000200053O00122O000400093O0026250004002D0001000900041B3O002D00010012373O00093O00041B3O0002000100041B3O002D000100041B3O000200012O00153O00017O00", GetFEnv(), ...);
