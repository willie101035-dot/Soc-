#FSM_counter

## 程式碼
```vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;

entity FSM_counter is
    generic(
        upper : integer := 9;
        lower : integer := 0
    );
    port(
        rst : in  STD_LOGIC;
        clk : in  STD_LOGIC;
        q   : out STD_LOGIC_VECTOR(3 downto 0)
    );
end FSM_counter;

architecture Behavioral of FSM_counter is
    type counter_state is (counter1, counter2);
    signal now_state, next_state : counter_state;

    signal Qn, Qn_d : integer := lower;
begin

    -- (1) State register (Sequential)
    process(clk, rst)
    begin
        if rst = '1' then
            now_state <= counter1;
        elsif rising_edge(clk) then
            now_state <= next_state;
        end if;
    end process;

    process(now_state, Qn)
    begin
        next_state <= now_state;

        case now_state is
            when counter1 =>
                if Qn >= upper then
                    next_state <= counter2;
                end if;

            when counter2 =>
                if Qn <= lower then
                    next_state <= counter1;
                end if;
        end case;
    end process;

    process(clk, rst)
    begin
        if rst = '1' then
            Qn <= lower;
        elsif rising_edge(clk) then
            Qn <= Qn_d;
        end if;
    end process;

    process(now_state, Qn)
    begin
        Qn_d <= Qn;

        case now_state is
            when counter1 =>
                if Qn >= upper then
                    Qn_d <= Qn - 1;
                else
                    Qn_d <= Qn + 1;
                end if;

            when counter2 =>
                if Qn <= lower then
                    Qn_d <= Qn + 1;
                else
                    Qn_d <= Qn - 1;
                end if;
        end case;
    end process;

    q <= std_logic_vector(to_unsigned(Qn, 4));

end Behavioral;

```
## 波形圖
<img width="922" height="578" alt="image" src="https://github.com/user-attachments/assets/27c58ced-f860-4d5d-9803-c4aec2f2cd46" />

## breakdown
<img width="481" height="356" alt="image" src="https://github.com/user-attachments/assets/2aa178f1-8c86-48f4-8dad-0731d5038b4e" />

## AOV
<img width="1036" height="406" alt="image" src="https://github.com/user-attachments/assets/9bfb208b-cdf6-41d4-b1b7-c99e54cbd109" />
##架構圖
<img width="1324" height="551" alt="image" src="https://github.com/user-attachments/assets/a01bed57-7d57-4acc-8527-47a4f090d0da" />
##MSC
<img width="1187" height="721" alt="image" src="https://github.com/user-attachments/assets/e26946da-819d-48d5-b8fc-e51f4061d846" />
