function HidRead( data )
	-- This uses the bitwise shortcut method of decoding a quadrature signal found at;
	-- http://www.parallax.com/dl/docs/cols/nv/vol1/col/nv8.pdf
	
	-- Set some variables we'll use later
	envTable = _G
	envTable = getfenv(1)
	if envTable.prev == nil then envTable.prev = "" end
	if envTable.prevBits == nil then envTable.prevBits = 0 end

	local s=""
	local index=0
	local curBits = envTable.prevBits

	if data then	
		-- Check each byte in the string.  This comes in the format XX XX XX XX XX XX
		for b in string.gfind(data, ".") do
			-- For the GPWiz 32, the first two bytes are always 77 77.  The third byte contains info we're interested in
			-- namely 01 == Quadrature A and 02 == Quadrature B
			if index == 2 then				
				curBits = math.band(string.byte(b),3)
			end
			-- Concatenate this bit back onto the data string
			s =s .. string.format("%02X ", string.byte(b))
 			index = index+1
		end
	end
	-- Check to see if our current data string is the same as the last, we don't want to do anything if it is.
	if s ~= envTable.prev then
		-- Bitshift our current byte one to the right so we can compare previous B with current A
		shifted = math.bshiftr(curBits,1)
		result = math.bxor(envTable.prevBits, shifted)
		-- Now we need to filter our result to only the second bit
		andResult = math.band(result, 1)
		if andResult == 0 and curBits ~= envTable.prevBits then
			gir.TriggerEvent("Quadrature Increased", 18, 0)
		elseif andResult == 1 and curBits ~= envTable.prevBits then
			gir.TriggerEvent("Quadrature Decreased", 18, 0)
		end
		gir.TriggerEvent(s, 239)
	end
	envTable.prevBits = curBits
	envTable.prev = s

end
